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

# 顶层设计与系统安全

#### 山东大学（威海） 2020级 数据科学与人工智能实验班

#### 讲解人：任鹏飞

#### 2022 年 1 月 20 日

幻灯片在 https://github.com/lie-flat/slides 仓库

---

# 全站不跨域，在同一个域名下
不跨域

我们的网站前端、后端、JupyterHub 全部在一个子域名 ( hub.kxxt.tech ) 下

没有采用多个子域名的方式（比如：前端，hub.kxxt.tech, 后端,api.kxxt.tech, JupyterHub, jupyter.kxxt.tech）

之所以这样设计是因为跨域请求存在一些缺点。

其中一个缺点就是，对于几乎每一次跨域请求，浏览器都要在请求前先发送一个 `OPTION` 请求，这增多了对服务器请求的数量，而几乎没有带来任何好处。

我们整个网站是一个整体，如果分散在多个子域名上，就显得不那么统一。

---

# 资源分配
Resource Allocation

宿主机是 2 核 4 G 的轻量云主机，资源比较有限

- 我们为每个 Jupyter Lab 实例设置内存上限为 0.9 G
- 设置最多同时有四个 Jupyter Lab 实例运行
- 为了防止用户过多，注册时需要邀请码

---

# 开箱即用
Out of the box

- 一般来说，在使用 Python 做数据分析时，文件的开头总是充斥着一大堆形式感满满的 `import` s
- 在使用我们的 JupyterHub 进行数据分析的时候，你几乎不用 `import` 任何东西，我们已经为你提前配置好了。
- 甚至数据库连接我们也已经提前为你配置好了，你只需要调用 `sql("SELECT a FROM b;")` 就可以直接调取数据，完全不用关心数据库的用户名、密码之类的东西。
- 我们开发的 CFPS Shell 也已经预先加载，它提供了非常方便的调取数据的方式。
- 另外，我们提前为 ECharts 做了优化，显示图表之前不用去调用 `chart.load_javascript()`, 也不会出现刷新一下页面，图表就全都不见了的情况。


---

# 数据库安全
Database Security

我们为存放 CFPS 数据的 MySQL 数据库合理的配置了用户。

在 Docker 容器内，只能使用 jupyterhub 这一受限账户访问数据库, 我们只授予了 SELECT 权限，有效的防止了 JupyterHub 用户篡改原始数据。

```sql
# 下面的 172.17.0.% 为 Docker 容器在 Docker 网卡上的 IP, 请根据实际情况设置
CREATE USER 'jupyterhub'@'172.17.0.%' IDENTIFIED BY 'SUPER-secret_PlavsW0r1d';
# 授予在 cfps 数据库上的 SELECT 权限（只读，防止用户篡改数据）
GRANT SELECT ON cfps.* TO 'jupyterhub'@'172.17.0.%';
FLUSH PRIVILEGES;
```

---

# 用户信息安全
User Data Security

使用标准的 OAuth2 方案实现 JupyterHub 的登录认证，确保用户的信息安全。

数据库中不明文存储密码，而是存储密码的哈希，即使数据库被拖库，黑客也无法得知用户的密码，这是现代互联网从业者所必须做到的。

全站使用 HTTPS 加密传输，防止有人在中间窃取/篡改信息。

---

# 宿主机安全
Host Security

Docker 容器内只能访问到主机的部分端口（比如MySQL 数据库和 JupyterHub API），防止黑客从容器内黑入主机



