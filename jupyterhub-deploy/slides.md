---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
fonts:
  mono: 'Cascadia Code'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# persist drawings in exports and build
drawings:
  persist: false
---

## CFPS JupyterHub 大作业讲解

# 部署

#### 山东大学（威海） 2020级 数据科学与人工智能实验班

#### 讲解人：任鹏飞

#### 2022 年 1 月 20 日

幻灯片在 https://github.com/lie-flat/slides 仓库

---

# 讲解目标
Goal

读者能够把我们的整套软件系统部署在自己的服务器上

---

# 前提条件
Prerequisites

假设你在使用 Linux 系统，已经安装好了 Anaconda、Docker 和 Nginx,还有 Node.js 和 Yarn.  假设你熟悉基本的 Linux 操作

我们的服务器使用的是 Debian GNU/Linux 10 (buster) x86_64, 如果您使用其他发行版，您可能需要根据实际情况修改部分命令。

另外，你需要按照大作业第一部分的教程配置好 MySQL 数据库。

```
       _,met$$$$$gg.          kxxt@kxxtserver
    ,g$$$$$$$$$$$$$$$P.       ---------------
  ,g$$P"     """Y$$.".        OS: Debian GNU/Linux 10 (buster) x86_64
 ,$$P'              `$$$.     Host: CVM 3.0
',$$P       ,ggs.     `$$b:   Kernel: 4.19.0-11-amd64
`d$$'     ,$P"'   .    $$$    Uptime: 10 days, 2 hours, 26 mins
 $$P      d$'     ,    $$P    Packages: 833 (dpkg), 6 (snap)
 $$:      $$.   -    ,d$$'    Shell: bash 5.0.3
 $$;      Y$b._   _,d$P'      Terminal: /dev/pts/0
 Y$$.    `.`"Y$$$$P"'         CPU: Intel Xeon Platinum 8255C (2) @ 2.499GHz
 `$$b      "-.__              GPU: Cirrus Logic GD 5446
  `Y$$                        Memory: 637MiB / 3818MiB
   `Y$$.
     `$$b.
       `Y$$b.
          `"Y$b._
              `"""
```

---

# 安装
Installation

首先，把我们的 Git 仓库克隆到本地，我们假设你把仓库放在了 `~/cfps-jupyterhub`

然后 `cd` 进去，运行：

```bash
conda env create -f environment.yml
conda activate cfps
pip install -r requirements.txt
cd hub-login
yarn install
cd ..
```

---

# 应用程序配置
App Configuration

## 环境变量配置

编辑 `.env` 文件，写入需要配置的环境变量

```bash
INVITE_CODES=邀请码，用;隔开，用户注册的时候需要
API_RATE_LIMIT=API调用频率上限，比如"30/minute"
API_SECRET_KEY=后端密钥
API_HUB_CLIENT_ID=JupyterHub客户端ID
API_HUB_CLIENT_SECRET=JupyterHub客户端密钥
APP_DOMAIN_NAME=部署域名
DOCKER_INTERFACE_IP=Docker 网卡 IP
APP_PROFILE=部署环境，可选值：dev(开发环境), prod(生产环境)
```

然后运行 `source load_env_vars.sh` 将 `.env` 文件中的环境变量加载到当前 shell 中

---

# 应用程序配置
App Configuration

## 配置存储

我们创建两个 docker volume，一个用来存储只读的共享数据，另一个用来存储所有用户都可读写的数据。

> 注：每个用户的私有存储将会自动创建，无需手动配置

```bash
mkdir -p /home/$USER/jupyterhub-data/{data,shared}
docker volume create --driver local -o o=bind -o type=none -o device="/home/$USER/jupyterhub-data/data" cfps-common-data
docker volume create --driver local -o o=bind -o type=none -o device="/home/$USER/jupyterhub-data/shared" cfps-team-shared
```

---
layout: two-cols
---

# 应用程序配置
App Configuration

## 配置存储

然后将大作业的 [第一部分](https://github.com/lie-flat/cfps-analyze) 的代码和数据复制到 `/home/$USER/data` 中, 目录结构应该如下：

其中，dataset 目录的结构应该根据 https://github.com/lie-flat/cfps-analyze/blob/master/README.MD 进行配置

根据相关保密要求，我们不能提供 CFPS 数据集文件，请从官方渠道申请并下载。

此 Repo 的 `cfps_dvapis` 文件夹和 `on_container_start_up.sh` 也应该被复制到 `/home/$USER/jupyterhub-data/data` 中

::right::

```
/home/$USER/jupyterhub-data/data
├── cfps-analyze
│   ├── dataset
│   ├── process
│   ├── docs
│   ├── .gitignore
│   ├── .gitmodules
│   ├── .whitesource
│   ├── LICENSE
│   ├── README.MD
│   └── requirements.txt
├── cfps_dvapis
|   └── ...
├── notebooks
|   └── ...
└── on_container_start_up.sh
```

注意要赋予 `on_container_start_up.sh` 可执行权限

这个 Repo 的 notebooks 目录下是示例笔记本文件，也应当把这个目录拷贝过去。

---

# 系统配置
System Configuration

（假设你所使用的系统用户已在 `docker` 组中，如果没有，请运行：`sudo usermod -aG docker $USER`） 提前下载好我制作的容器镜像

```bash
docker pull rsworktech/cfps-notebook:latest
```

允许 Docker 容器访问主机的 8081 端口和部署在 3306 端口的 MySQL

```bash
sudo iptables -A INPUT -i docker0 -p tcp -m tcp --dport 8081 -j ACCEPT
sudo iptables -A INPUT -i docker0 -p tcp -m tcp --dport 3306 -j ACCEPT
```

持久化 iptables 的设置，防止重启系统之后上面的设置失效

如果没有安装 `iptables-persistent`, 则先运行 `sudo apt install iptables-persistent`

```bash
sudo dpkg-reconfigure iptables-persistent
```

---

# 系统配置
System Configuration

生成 Nginx 配置


> ##### 注意
> 
> 请确保你在当前的 Shell 运行过 `source load_env_vars.sh`, 如果不确定的话，请运行一遍



```bash
conda activate cfps
python generate_nginx_conf.py
```


安装 Nginx 配置并启用

```bash
sudo cp hub.kxxt.tech.conf /etc/nginx/sites-available/
sudo ln -s /etc/nginx/sites-available/hub.kxxt.tech.conf /etc/nginx/sites-enabled/hub.kxxt.tech.conf
```

---


# 系统配置
System Configuration

配置 SSL 证书 (假设你已经装好了 `certbot`)

<div style="height: 370px;overflow: auto;">

```bash
(cfps) kxxt@kxxtserver:~/cfps-jupyterhub$ sudo certbot
Saving debug log to /var/log/letsencrypt/letsencrypt.log

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
5: hub.kxxt.tech
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 5
Requesting a certificate for hub.kxxt.tech

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/hub.kxxt.tech/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/hub.kxxt.tech/privkey.pem
This certificate expires on 2022-04-14.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for hub.kxxt.tech to /etc/nginx/sites-enabled/hub.kxxt.tech.conf
Congratulations! You have successfully enabled HTTPS on https://hub.kxxt.tech
```

</div>

---

# 数据库配置
Database Configuration


为 JupyterHub 的用户创建一个数据库账户

```bash
sudo mysql
```


```sql
# 下面的 172.17.0.% 为 Docker 容器在 Docker 网卡上的 IP, 请根据实际情况设置
CREATE USER 'jupyterhub'@'172.17.0.%' IDENTIFIED BY 'SUPER-secret_PlavsW0r1d';
# 授予在 cfps 数据库上的 SELECT 权限（只读，防止用户篡改数据）
GRANT SELECT ON cfps.* TO 'jupyterhub'@'172.17.0.%';
FLUSH PRIVILEGES;
```

---

# 启动应用程序
Launch

## 调试模式和生产模式

在调试模式下，程序会因为源代码文件的改动而自动重新加载，比较适合开发时使用。

编辑 `.env` 文件， 将 `APP_PROFILE` 改为 `dev` 或 `prod`

我们推荐您使用 `tmux` 来管理会话 (`sudo apt install tmux`)

~~当然你用 `screen` 也可以~~。

新开一个 tmux 会话，分成三块 pane：

```bash
tmux
```

---

# 启动应用程序
Launch

然后按下组合键

```
C-b %  # 水平分屏
C-b "  # 垂直分屏
```

> 这里 `C-b %` 指先按下 <kbd>Ctrl</kbd> + <kbd>B</kbd>, 然后输入 % （即按下 <kbd>Shift</kbd> + <kbd>5</kbd>）
> 本文出现的其他的按键组合同理

在三个 pane 中都执行如下操作, 方法如下

```bash
C-b :
setw synchronize-panes on # 此处有 Tab 补全，不用傻傻的一个字一个字的敲。。。
```

然后执行

```bash
cd cfps-jupyterhub  # 进入到此 Repo 的目录
conda activate cfps  # 激活 conda 环境
source load_env_vars.sh  # 加载环境变量
```

---

# 启动应用程序
Launch

然后退出同步执行模式：

```bash
C-b :
setw synchronize-panes off # 此处有 Tab 补全，不用傻傻的一个字一个字的敲。。。
```

如果你修改过 .env, 就重新生成 Nginx 配置文件并部署

```
python generate_nginx_conf.py
sudo rm /etc/nginx/sites-available/hub.kxxt.tech.conf
sudo cp hub.kxxt.tech.conf /etc/nginx/sites-available/hub.kxxt.tech.conf
sudo systemctl reload nginx
```

然后重新配置 SSL,选择好站点之后， 选 1, 重新安装即可，无需 Renew

```bash
sudo certbot
Successfully deployed certificate for hub.kxxt.tech to /etc/nginx/sites-enabled/hub.kxxt.tech.conf
Congratulations! You have successfully enabled HTTPS on https://hub.kxxt.tech 
```

---


# 启动应用程序
Launch

然后运行 JupyterHub, 把日志保存在 `jupyterhub.log`

```bash
jupyterhub -f jupyterhub_config.py &> jupyterhub.log
```

然后切换到另一个 pane (<kbd>Ctrl</kbd> + <kbd>B</kbd>, 然后按方向键)

运行后端服务, 把日志输出到 `backend.log`：

```bash
python server.py &> backend.log
```


---

# 启动应用程序
Launch

然后切换到最后一个 pane，配置一下前端：

如果是调试模式，就启动 Next.js Server：

```bash
cd hub-login
yarn dev
```

如果是生产模式，把静态网站构建出来就可以了

```bash
cd hub-login
yarn build
```

好了，现在整个程序应该都跑起来了！

按下 <kbd>Ctrl</kbd> + <kbd>B</kbd> , 然后按下 <kbd>d</kbd>, 可以与当前的 tmux 会话分离

以后使用 `tmux attach` 命令可以重新进入原来的 tmux 会话

---

# 部署完成
Deployed

好了，部署工作完成了，你可以尽情的享受部署的 JupyterHub 了。