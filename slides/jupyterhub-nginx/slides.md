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

# 整体网络设计与 Nginx 配置

#### 山东大学（威海） 2020级 数据科学与人工智能实验班

#### 讲解人：任鹏飞

#### 2022 年 1 月 19 日

幻灯片在 https://github.com/lie-flat/slides 仓库

---
layout: two-cols
---

# 目录结构

<div style="width: 90%;">

```
cfps-jupyterhub
├── .env # 环境变量存储文件
├── dev.template.conf # 开发模式的配置模板
├── generate_nginx_conf.py # 生成 Nginx 配置的脚本
├── load_env_vars.sh # 加载 .env 文件的脚本
└── prod.template.conf # 生产模式的配置模板
```

</div>

::right::

# 整体划分

- 外部网络 / 内部网络 / Docker 网络
- 前端静态页面/前端服务器
- 后端服务器
- JupyterHub 的代理和代理的 RESTful API
- JupyterHub 的 Hub
- Docker 容器内/外
- Jupyter Lab
- 开发环境/生产环境
- 防火墙（iptables 规则）

---

# 整体设计 (因为我没学过计网，如有偏差还请谅解)

<img src="/arch.png" style="height: 96%;"/>

---

# 环境配置
_

环境变量写在 `.env` 里面

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

使用 `source load_env_vars.sh` 加载环境变量到当前会话

```bash
#!/bin/sh
# file: load_env_vars.sh
set -a
source ./.env
set +a
```

---

# Nginx 配置模板

```nginx
# top-level http config for websocket headers
# If Upgrade is defined, Connection = upgrade
# If Upgrade is empty, Connection = close
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
```

配置一些 WebSocket 相关的 Header, 因为 JupyterHub 会用到他们

---

# Nginx 配置模板
Basic

```nginx
server {
    listen 80;
    server_name {domain_name};
    charset UTF-8;
    root {root_dir}/hub-login/out;
    index index.html;
    gzip on;
    gzip_buffers 32 4K;
    gzip_comp_level 6;
    gzip_min_length 100;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";
    gzip_types text/plain text/css text/javascript application/javascript application/json application/x-javascript text/xml application/xml application/xml+rss;
```

- 设置域名，字符集，根目录，首页，开启 gzip 压缩
- 这里设置 listen 80 而不是 433 是因为我们的 SSL 是通过 certbot 自动配置的
- 我们只需要写 http 的配置即可， 之后我们运行 certbot 的时候它会修改配置，自动转换为 https

---

# Nginx 配置模板
JupyterHub

```nginx
location /jupyter/ {
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header Host $host;
  # websocket headers
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection $connection_upgrade;
  proxy_set_header X-Scheme $scheme;
  proxy_buffering off;
  proxy_pass http://127.0.0.1:{hub_port}/jupyter/;
}
```

这段配置 JupyterHub 官方有提供，直接照搬即可，只需设置一下 url.

---

# Nginx 配置模板
Backend

```nginx
location /api/ {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    proxy_pass http://127.0.0.1:{api_port}/;
}
```

简简单单手搓一个反向代理配置，有啥好讲的，Skip

---
layout: two-cols
---

# Nginx 配置模板
前端


<div style="width:90%;">

### 开发环境

```nginx
location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    # websocket headers
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header X-Scheme $scheme;
    proxy_buffering off;
    proxy_pass http://127.0.0.1:3000/;
}
```

一个简简单单的反向代理，另外配置一下 WebSocket, 模块热重载要用到。

</div>

::right::

<div style="width:90%;">

## 生产环境
  
```nginx
# https://stackoverflow.com/questions/19515132/nginx-cache-static-files
## All static files will be served directly.
location ~* ^(?!/jupyter/).+\.(?:css|cur|js|jpe?g|gif|htc|ico|png|html|xml|otf|ttf|eot|woff|woff2|svg)$ {
  access_log off;
  expires 30d;
  add_header Cache-Control public;
  ## No need to bleed constant updates. Send the all shebang in one
  ## fell swoop.
  tcp_nodelay off;
  ## Set the OS file cache.
  open_file_cache max=3000 inactive=120s;
  open_file_cache_valid 45s;
  open_file_cache_min_uses 2;
  open_file_cache_errors off;
}
location / {
  try_files $uri $uri/ =404;
}
```

配置静态资源的缓存。然后是一个经典的静态网站的配置，注意前面已经把 `root` 设置好了
</div>

---
layout: two-cols
---

# Nginx 配置生成脚本
generate_nginx_conf.py

<div style="width:105%;">

```python
from backend.config import port, APP_PROFILE, DOMAIN_NAME
import pathlib
import sys
root = pathlib.Path(__file__).parent.resolve()
if len(sys.argv) > 1:
    mode = sys.argv[1]
else:
    mode = APP_PROFILE
with open(f"{mode}.template.conf", "r", encoding='utf-8') as f:
    template = f.read()
with open(f"{DOMAIN_NAME}.conf", "w", encoding='utf-8') as f:
    f.write(template
            .replace("{api_port}", str(port))
            .replace("{domain_name}", DOMAIN_NAME)
            .replace("{hub_port}", str(8000))
            .replace("{root_dir}", str(root))
            )
print("Success!")
```

</div>

::right::

<div style="width:80%;margin-left: 10%;">

- 从后端设置中读取后端端口，运行模式和域名
- 得到这个文件所在的路径作为 `root_dir`
- 如果有参数，则使用参数作为运行模式，否则使用 `APP_PROFILE`
- 读取对应的模板文件
- 将模板文件中的变量替换掉，输出结果文件
- 打印成功信息，结束

</div>

