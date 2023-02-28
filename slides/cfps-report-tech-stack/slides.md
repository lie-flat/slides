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
# some information about the slides, markdown enabled
info: |
  ## CFPS 数据可视化网站技术栈
# persist drawings in exports and build
drawings:
  persist: false
---

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

# CFPS 数据可视化网站技术栈

#### 山东大学（威海）

#### 2020级 数据科学与人工智能实验班

#### 讲解人：任鹏飞

#### 2021 年 12 月 8 日

---
layout: two-cols
---
# Javascript 包管理器
Yarn instead of npm

我们选用 Yarn 作为 JavaScript 的包管理器，没有使用默认的 npm。因为 Yarn 安装依赖速度更快一些，

Yarn 是一个软件包管理器，还可以作为项目管理工具。无论你是小型项目还是大型单体仓库（monorepos），无论是业余爱好者还是企业用户，Yarn 都能满足你的需求。

::right::

![Yarn](/slides.assets/yarn.svg)

![See the source image](/slides.assets/yarn-vs-npm-in-2019-speed-comparison-in-a-middle-sized-apps.jpg)

<style>
img {
margin-left: 2rem;
}
</style>

---
layout: two-cols
---

# 前端框架

Vue3.0

我们采用 Vue 3 作为项目的框架，使用 Vue3.0 构建出纯静态的网页。

通过使用 Vue 3, 我们可以合理的将网站的各个部分划分成 **SFC(Single File Component, 单文件组件)**，借此来实现 **关注点分离(Separate of concerns)**。

同时，划分多个 SFC 也有助于我们小组进行合作，降低整个工程的复杂性。

因为这一部分作业的工程复杂性不是很高，我们直接使用 <code style="background: yellow;">JavaScript</code> 编程语言来开发，并没有采用 <code style="background: #3178c6;color: white;">Typescript</code> 等引入静态类型检查的语言。

::right::

![Vue.js](/slides.assets/logo.png)

---
layout: two-cols
---

# CSS 框架

Bulma

Bulma 是一个使用 Sass 语言编写的模块化的响应式 CSS 框架，基于 flexbox 技术。

- 借助 Bulma, 我们可以很轻松的做出响应式的网页。
- 由于其模块化的特性，我们可以很大程度上裁剪输出的 CSS 文件的大小

Bulma is a free, open source framework that provides ready-to-use frontend components that you can easily combine to build responsive web interfaces.

::right::

![Bulma: Free, open source, and modern CSS framework based on Flexbox](/slides.assets/bulma-logo.png)

<style>
img {
margin-left: 2rem;
}
</style>


---
layout: two-cols
---
# CSS 扩展
Sass

我们使用 SCSS 来编写样式表（Stylesheet）

SCSS 在 CSS 的基础上添加了很多强大易用的功能

（Sass 有两种语法，一种是 SassScript，采用缩进语法，与 CSS 不兼容，另一种是 SCSS，与 CSS 兼容）

::right::

![Sass](/slides.assets/logo-b6e1ef6e.svg)


---
layout: two-cols
---

# 数据可视化框架
echarts & vue-echarts

我们使用 ECharts 作为数据可视化的框架。

因为我们的数据可视化主要就是画一些图表，并不需要深入的去定制数据可视化，所以我们没有采用类似 D3.js 这样的库

与此同时，我们使用 `vue-echarts` 来在 vue3 中更方便使用 echarts

::right::

![echarts logo](/slides.assets/logo-16388091078794.png)

![image-20211207005239608](/slides.assets/image-20211207005239608.png)

<style>
img {
	margin-left:1rem;
}
</style>

---
layout: two-cols
---
# CSS 动画框架

Animate.css

<img src="/slides.assets/b.gif" alt="b" style="zoom:80%;" />

::right::

<div>

<img src="/slides.assets/animatecss-opengraph.jpg" alt="animatecss-opengraph.jpg" style="zoom:80%;" />

我们使用 Animate.css 作为动画效果的框架，<del>（其实也没用到多少里面的效果）</del>

除此之外，网站上的部分动画效果是我们手动编写的。

</div>

<style>
div {
margin-left: 2rem;
}
</style>


---
layout: two-cols
---
# HTTP Server
HTTP 服务器

![image-20211206220732601](/slides.assets/image-20211206220732601.png)

::right::

我们使用 `Nginx` 进行静态网页的部署

Nginx 是一个高性能 HTTP & 反向代理服务器。

nginx [engine x] is an HTTP and reverse proxy server, a mail proxy server, and a generic TCP/UDP proxy server, originally written by Igor Sysoev. For a long time, it has been running on many heavily loaded Russian sites including Yandex, Mail.Ru, VK, and Rambler. According to Netcraft, nginx served or proxied 22.36% busiest sites in November 2021. Here are some of the success stories: Dropbox, Netflix, Wordpress.com, FastMail.FM.


---
layout: two-cols
---

# SSL
SSL & HTTPS

![Let's Encrypt](/slides.assets/letsencrypt-logo-horizontal.svg)

::right::

我们采用 Let's Encrypt 提供的免费 SSL 证书来为网站提供加密的 https 连接。

Let’s Encrypt 是一家免费、开放、自动化的证书颁发机构（CA），为公众的利益而运行。它是一项由 Internet Security Research Group（ISRG）提供的服务。

---
layout: two-cols
---

# Slidev
Slidev 

Slidev 是一个基于  Vue 3, Windi CSS 和 Vite 的 Markdown + Vue 幻灯片框架。

你现在所观看的这个幻灯片就是使用 Slidev 制作的

借助 Slidev, 我们可以更方便的在幻灯片中插入代码和自定义组件，使用简洁优雅的 Markdown 语法来制作幻灯片。

你的下一个 PPT , 何必要使用 PowerPoint?

https://cn.sli.dev

::right::

![Slidev](/slides.assets/logo-title.png)

