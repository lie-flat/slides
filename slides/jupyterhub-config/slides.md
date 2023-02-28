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

# JupyterHub 配置

#### 山东大学（威海） 2020级 数据科学与人工智能实验班

#### 讲解人：任鹏飞

#### 2022 年 1 月 18 日

幻灯片在 https://github.com/lie-flat/slides 仓库

---

配置文件： `cfps-jupyterhub/jupyterhub_config.py`

```python
# Configuration file for jupyterhub.
import os
import sys
import pathlib

sys.path.append(str(pathlib.Path(__file__).parent.resolve()))

from backend.config import HUB_CLIENT_ID, HUB_CLIENT_SECRET, port
```

首先我们从另一个配置文件中导入一些配置

- port: FastAPI 后端的运行端口
- HUB_CLIENT_ID: JupyterHub 的客户端 ID
- HUB_CLIENT_SECRET: JupyterHub 的客户端密钥

---

```python
c = get_config()

c.JupyterHub.active_server_limit = 4
c.JupyterHub.cleanup_servers = True
c.JupyterHub.concurrent_spawn_limit = 3
c.JupyterHub.cookie_max_age_days = 7
c.JupyterHub.spawner_class = 'dockerspawner.DockerSpawner'
```

- 然后设置最多有四个活跃的实例，
- 设置在 JupyterHub 关闭时，停止所有的实例，
- 最多同时启动三个实例，
- 设置 cookie 的最大存活时间为 7 天，
- 使用 DockerSpawner 启动实例

---

```python
c.JupyterHub.bind_url = 'http://127.0.0.1:8000/jupyter'  # 因为有 nginx, 所以代理只监听 127.0.0.1:8000 即可

DOCKER_INTERFACE_IP = os.environ.get('DOCKER_INTERFACE_IP', '172.17.0.1')

c.JupyterHub.hub_ip = DOCKER_INTERFACE_IP  # 设置为 Docker 虚拟网卡的 IP
c.DockerSpawner.hub_ip_connect = DOCKER_INTERFACE_IP
c.DockerSpawner.hub_connect_url = f"http://{DOCKER_INTERFACE_IP}:8081/"
print(c.JupyterHub.hub_ip)
```

- 设置 JupyterHub 自身绑定的地址
- 从环境变量中获取 Docker 虚拟网卡的 IP
- 把 JupyterHub 绑定的 IP 设置为 Docker 虚拟网卡的 IP
- 设置 Docker 容器中连接 JupyterHub API 的 IP 地址及端口

---

```python
notebook_dir = os.environ.get(
    'DOCKER_NOTEBOOK_DIR') or '/home/jovyan'
c.DockerSpawner.notebook_dir = notebook_dir
c.DockerSpawner.image = "rsworktech/cfps-notebook:latest"
c.DockerSpawner.volumes = {
    'jupyterhub-user-{username}': notebook_dir,
    'cfps-common-data': {'bind': '/home/jovyan/data',
                         'mode': 'ro'},
    'cfps-team-shared': '/home/jovyan/team-shared'
}
c.Spawner.mem_limit = '0.9G'
```
- 从环境变量中获取 Jupyter Lab 的运行目录，默认即家目录
- 设置使用的 Docker Image 为我上传的 `cfps-notebook`
- 设置存储，把一个卷挂在 `notebook_dir` 上，这个卷是每个用户独享的
- 另外，把数据卷挂在 `/home/jovyan/data` 上，这个卷是共享的，但是用户只有读的权限
- 其次，把团队共享的卷挂在 `/home/jovyan/team-shared` 上，这个卷是共享的，所有用户都有读写权限
- 然后设置内存上限为 0.9G

---

```python
c.JupyterHub.authenticator_class = "oauthenticator.generic.GenericOAuthenticator"
c.GenericOAuthenticator.oauth_callback_url = '/jupyter/oauth_callback'
c.GenericOAuthenticator.client_id = HUB_CLIENT_ID
c.GenericOAuthenticator.client_secret = HUB_CLIENT_SECRET
c.GenericOAuthenticator.login_service = '统一身份认证服务'
c.GenericOAuthenticator.userdata_url = f'http://127.0.0.1:{port}/user-data'
c.GenericOAuthenticator.token_url = f'http://127.0.0.1:{port}/token'
c.GenericOAuthenticator.authorize_url = '/login'
c.GenericOAuthenticator.username_key = 'username'
```

- 设置使用的认证方式为 `GenericOAuthenticator`
- 设置回调地址为 `/jupyter/oauth_callback`， 这是用户在我们的后端认证完成后，会跳转到的地址
- 设置 client_id 和 client_secret
- 设置登录服务的名称为 `统一身份认证服务`
- 设置用户数据的获取地址为我们后端相应的 API 的地址
- 设置 token 的获取地址为我们后端相应的 API 的地址
- 设置认证的页面为我们前端页面的地址，即要求用户登录的地址
- 设置在 `user-data` API 返回的数据中，用户名的键为 `username`

---

```python
DATABASE_PORT = os.environ.get('DATABASE_PORT', '3306')
DATABASE_USER = os.environ.get('DATABASE_USER', 'jupyterhub')
DATABASE_PASSWORD = os.environ.get('DATABASE_PASSWORD', 'SUPER-secret_PlavsW0r1d')
DATABASE_NAME = os.environ.get('DATABASE_NAME', 'cfps')

c.Spawner.environment = {
    'LANG': 'zh_CN.utf8',
    'DATABASE_HOST': DOCKER_INTERFACE_IP,
    'DATABASE_PORT': DATABASE_PORT,
    'DATABASE_USER': DATABASE_USER,
    'DATABASE_PASSWORD': DATABASE_PASSWORD,
    'DATABASE_NAME': DATABASE_NAME,
    'CFPS_SHELL_DATA_ROOT': '/home/jovyan/data/cfps-analyze/',
}
```

- 设置数据库服务的 IP 地址为 Docker 虚拟网卡的 IP
- 从环境变量中读取数据库的端口、用户名、密码、数据库名称，再通过环境变量传递给 Docker 容器
- 设置语言偏好为中文
- 设置 `CFPS_SHELL_DATA_ROOT` , 这样我们开发的 CFPS Shell 就知道去哪里加载数据了。

---

```python
c.JupyterHub.load_roles = [
    {
        "name": "jupyterhub-idle-culler-role",
        "scopes": [
            "list:users",
            "read:users:activity",
            "read:servers",
            "delete:servers",
        ],
        "services": ["jupyterhub-idle-culler-service"],
    }
]
c.JupyterHub.services = [
    {
        "name": "jupyterhub-idle-culler-service",
        "command": [
            sys.executable,
            "-m", "jupyterhub_idle_culler",
            "--timeout=3600",
        ],
    }
]
```

配置 idle-culler 服务, 给予它适当的权限，将它设置为 JupyterHub 的一个 service, 用户一小时不活动就会被清理掉
