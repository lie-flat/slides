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

# OAuth2 与 登录/注册

#### 山东大学（威海） 2020级 数据科学与人工智能实验班

#### 讲解人：任鹏飞

#### 2022 年 1 月 19 日

幻灯片在 https://github.com/lie-flat/slides 仓库

---

# JupyterHub 的登录方式
——

JupyterHub 支持多种登录认证的方式， 比如

- 默认的 PAM Authenticator, 用户需要是服务器上的本地用户
- <span style="color:yellow;font-weight: bold;">OAuthenticator, OAuth2 认证</span>
- Dummy Authenticator, 用户名随意，只要密码正确就通过认证
- Native Authenticator, 一个 JupyterHub 官方提供的支持登录/注册的 Authenticator

我们采用的 OAuthenticator, JupyterHub 以 OAuth2 标准跟我们编写前后端进行交互来实现登录认证

另外，我们实现了一个使用邀请码的注册方式。

> 注意
> 
> - Authentication 是认证的意思，不是登录，登录是认证的一种特例。认证是确认用户的身份。
> - Authorization 是授权的意思, 跟 Authentication 不同，Authorization 是授予用户的权限，各位读者不要混淆概念。

---

# 流程

<img src='/oauth.png' style="height: 90%;"/>