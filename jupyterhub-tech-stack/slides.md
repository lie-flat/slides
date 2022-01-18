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

# 技术栈讲解

#### 山东大学（威海） 2020级 数据科学与人工智能实验班

#### 讲解人：任鹏飞

#### 2022 年 1 月 18 日

幻灯片在 https://github.com/lie-flat/slides 仓库

---

## 前端技术栈

#### Next.js

我们使用 Next.js 来生成静态（static）的网页,然后使用 Nginx 部署。

Next.js 是一个基于 React 框架的前端框架，提供了强大的 SSR (Server Side Rendering) 功能, 即使是我们的用户在浏览器里禁用了 JavaScript，我们的网页也能比较正常的显示（虽然体验会下降，但不会和一般的 React/Vue 项目那样禁用 JS 之后网站直接变成空白）。

#### SCSS 和 styled-components

一部分样式使用 SCSS 编写，也有一部分使用 styled-components。

#### Bulma CSS 框架

使用 `react-bulma-components`，它把 Bulma 的很多概念封装成了 React 组件。

#### 其他

- react-full-page: 实现全页滚动， react-lazy-load-image-component: 图片懒加载
- formik: 表单相关的库， axios: 用来发送请求
- fontawesome: 图标库

---

## 后端技术栈

#### FastAPI

FastAPI 是一个构建 RESTful API 的 Python 框架, 性能很好（相比与 django 和 flask, 当然无法和 C/C++/Rust/C#/Java 等语言写出来的框架相比）.

<img src="/x.png" width="500" >

---

## 后端技术栈

#### SQLite 和 SQLAlchemy

考虑到我们的机器性能并不是很强，大概也就只能支撑四五个用户同时使用 Jupyter Lab, 而且也用不上特别高级的数据库功能， 所以后端数据库用 SQLite 足矣。

SQLite 是一个非常小巧的数据库，所有的数据都存放在一个文件中。

SQLAlchemy 是一个 ORM 框架，它可以让我们更加方便地操作数据库。

#### Uvicorn

Uvicorn 是一个高速的 ASGI 服务器， FastAPI 官方教程里主要用的它，我们也就直接用它了，何况在上一页 PPT 上大家也看到了 uvicorn 是性能最强的 Python Web 框架。


#### Nginx

简简单单做个反向代理

#### slowapi

用来对用户的请求速度做限制（Rate Limit），防止别有用心的人尝试暴力破解密码/邀请码。

---

## JupyterHub 部署技术栈

_

### oauthenticator

配置 JupyterHub 的认证方式为 OAuth2，通过我们自己编写的后端 OAuth2 服务来进行认证。

### jupyterhub-idle-culler

停止闲置的 Jupyter Lab 实例，节约服务器资源。


### SQLite

采用 SQLite 数据库（JupyterHub 的默认配置）

### dockerspawner

配置 JupyterHub 通过 DockerSpawner 为用户分配 Jupyter Lab 实例，保证了用户的实例与宿主机的隔离

---

## JupyterHub 部署技术栈

为什么不用 Kubernetes?

- 我们的用户量没那么大，也不会朝着大规模的方向去发展。
- 我们只有一台服务器，没有同时处于一个内网的多台服务器，不具备使用 K8S 的优越条件
- K8S 的优点在于它的动态扩展和负载均衡上，而跟据第二条，我们都用不上
- 所以最终没有采用 K8S 方案

---

## 我们的自定义 Docker 镜像(Image)内所用技术栈
_

- 一些基础工具，如 git, vim, nano, wget
- Python 3.10, 最新版本的 Python
- conda 环境
- pyecharts, matplotlib, pandas, numpy, scipy, sympy 等常用的数据分析/可视化库
- SQLAlchemy 等用来连接数据库的库
- 最新版本的 Jupyter Lab
- Node.js & npm