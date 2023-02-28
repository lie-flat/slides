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

# 前端

#### 山东大学（威海） 2020级 数据科学与人工智能实验班

#### 讲解人：任鹏飞

#### 2022 年 1 月 20 日

幻灯片在 https://github.com/lie-flat/slides 仓库

---
layout: two-cols
---

# 目录结构

<div style="width:90%;">

## 组件子目录

```
hub-login/components
├── container.js
├── field-container.js
├── form-layout.js
├── icon-text.js
├── invite-code-icon.js
├── jupyter-icon.js
├── LazyImage.js
├── password-icon.js
├── simple-message.js
├── slide-layout.js
├── SlideLayout.module.scss
├── slides
│   ├── foot-slide.js
│   ├── head-slide.js
│   ├── space-travelling.js
│   └── SpaceTravelling.module.scss
├── styled-button.js
├── styled-error-message.js
├── styled-link.js
├── title.js
└── user-icon.js
```

</div>

::right::

<div style="width:90%;">

## 前端工程目录
```
hub-login
├── components/
├── next.config.js
├── package.json
├── pages
│   ├── _app.js
│   ├── _document.js
│   ├── index.js
│   ├── login.js
│   └── register.js
├── public
│   ├── favicon.ico
│   ├── icon-jupyter.svg
│   └── pictures
│       ├── cfps.gif
│       ├── data.webp
│       ├── docker.png
│       ├── jupyterlab.png
│       ├── oauth2.png
│       └── ootb.jpg
├── README.md
├── styles/globals.css
├── utils.js
└── yarn.lock
```

</div>

---
layout: section
---

# 主页及其相关组件

---

# 主页
pages/index.js

```js
const GlobalStyles = createGlobalStyle`
  body {
    background-color: #fafafa;
  }
`
```

- 设置全局背景色（styled-components 写法）
- import 一堆东西，我省略了

---

# 主页
pages/index.js

```js
export default function Home() {
    return (
        <>
            <Head><title>CFPS JupyterHub</title></Head>
            <GlobalStyles/>
            <FullPage duration={400}>
                <Slide><HeadSlide/></Slide>
                <Slide>
                    <SlideLayout direction="渐变方向" gradients='渐变色' imagePosition='图片位置'>
                        <LazyImage width={360} height={112.717} src="图片" alt="图片替换文字"/>
                        <SimpleMessage color='颜色' icon={图标} title="节标题">
                           内容
                        </SimpleMessage>
                    </SlideLayout>
                </Slide>
                <Slide>...</Slide>
                <Slide><FootSlide/></Slide>
            </FullPage>
        </>
    )
}

```

---

# 开头
components/slides/head-slide.js

```js
const IntroText = styled.p`margin-top: 1em;`
const ThinButton = styled(Button)`width: 7em;`
export default function HeadSlide() {
    return (
        <SpaceTraveling>
            <Heading renderAs='h1' size={1}>CFPS JupyterHub</Heading>
            <Heading renderAs='h2' size={4}>山东大学 (威海) 数据科学与人工智能实验班</Heading>
            <Heading renderAs='h2' size={5}>前端开发和数据库课程大作业</Heading>
            <Button.Group>
                <ThinButton color='warning' renderAs='a' href="https://github.com/lie-flat/cfps-jupyterhub">
                    <IconText size="sm" icon={faGithub}>Github</IconText>
                </ThinButton>
                <ThinButton color='danger' renderAs='a' href="#">
                    <IconText size="sm" icon={faBilibili}>Bilibili</IconText>
                </ThinButton>
            </Button.Group>
            <Button color='primary' renderAs='a' href="/jupyter" style={{width: '10em'}}>
                <IconText size='sm' icon={faPlay}>立即试用</IconText>
            </Button>
            <IntroText>还没想好？下滑查看介绍</IntroText>
        </SpaceTraveling>
    )
}
```

---

# 结尾
components/slides/foot-slide.js

```js
const HeadingWithMargin = styled(Heading)`margin-top: 1em;`
const ThinButton = styled(Button)`width: 8em;`
export default function FootSlide() {return (
        <SpaceTraveling>
            <Heading renderAs='h4' size={1}>心动不如行动</Heading>
            <Heading renderAs='h5' size={4}>不妨现在就去试试？</Heading>
            <Button.Group>
                <ThinButton color='warning' renderAs='a' href="https://github.com/lie-flat/cfps-jupyterhub">
                    <IconText size="sm" icon={faGithub}>Github</IconText>
                </ThinButton>
                <ThinButton color='danger' renderAs='a' href="#">
                    <IconText size="sm" icon={faBilibili}>Bilibili</IconText>
                </ThinButton>
            </Button.Group>
            <Button color='primary' renderAs='a' href="/jupyter" style={{width: '10em'}}>
                <IconText size="sm" icon={faPlay}>立即试用</IconText>
            </Button>
            <HeadingWithMargin renderAs='h5' size={4}>友情链接</HeadingWithMargin>
            <Button.Group>
                <ThinButton color='info' renderAs='a' href="https://www.isss.pku.edu.cn/cfps/">CFPS 官网</ThinButton>
                <ThinButton color='success' renderAs='a' href="https://kxxt.vercel.app">kxxt 的个人网站</ThinButton>
            </Button.Group>
        </SpaceTraveling>)}
```

---

# 开头和结尾的 SpaceTravelling 特效
components/slides/space-travelling.js

这个特效我是根据这篇教程制作的： https://github.com/chokcoco/iCSS/issues/148 ，在此对原作者表示感谢。

```js
const Group = () => (
    <div className={styles.group}>
        <div className={`${styles.item} ${styles.itemRight}`}/>
        <div className={`${styles.item} ${styles.itemLeft}`}/>
        <div className={`${styles.item} ${styles.itemTop}`}/>
        <div className={`${styles.item} ${styles.itemBottom}`}/>
        <div className={`${styles.item} ${styles.itemMiddle}`}/>
    </div>
)
export default function SpaceTraveling({children}) {
    return (
        <>
            <div className={styles.rootContainer}>
                <Container className={styles.content}>{children}</Container>
                <div className={styles.container}><Group/><Group/></div>
            </div>
        </>
    )
}
```

---
layout: two-cols
---

# 开头和结尾的 SpaceTravelling 特效
components/slides/SpaceTravelling.module.scss

<p style="width: 90%;">
具体的实现方法请参见：https://github.com/chokcoco/iCSS/issues/148 ，原作者讲的很详细，我不再赘述了。
</p>
::right::


<div style="height: 50ch;width: 110%;margin-left: -10%;overflow: scroll;">

```scss
@use "sass:math";

.rootContainer {
  background-color: black;
  width: 100%;
  height: 100%;
  display: grid;
  grid-template-columns: 1fr;
  grid-template-rows: 1fr;
  overflow: hidden;
}

.container {
  display: flex;
  grid-area: 1/1/2/2;
  position: relative;
  background: green;
  perspective: 4px;
  margin: auto;
  perspective-origin: 50% 50%;
  animation: hueRotate 20s infinite linear;
}

@keyframes hueRotate {
  0% {
    filter: hue-rotate(0);
  }
  100% {
    filter: hue-rotate(360deg);
  }
}

.group {
  position: absolute;
  border-radius: max(50vh, 50vw);
  width: 100vw;
  height: 100vh;
  left: -50vw;
  top: -50vh;
  transform-style: preserve-3d;
  animation: move 8s infinite linear;
}

.group:nth-child(2) {
  animation: move 8s infinite linear;
  animation-delay: -4s;
}

@keyframes move {
  0% {
    transform: translateZ(-50px) rotate(0deg);
  }

  50% {
    transform: translateZ(0px) rotate(180deg);
  }

  100% {
    transform: translateZ(50px) rotate(360deg);
  }
}

@keyframes fade {
  0% {
    opacity: 0;
  }
  25%, 60% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

.item {
  position: absolute;
  opacity: 1;
  width: 100%;
  height: 100%;
  animation: fade 8s infinite linear;
  background-size: cover;
  animation-delay: 0s;
}

@function randomNum($max, $min: 0, $u: 1) {
  @return ($min + random($max)) * $u;
}

@function randomColor() {
  @return rgb(randomNum(255), randomNum(255), randomNum(255));
}


@function shadowSet($maxWidth, $maxHeight, $count) {
  $shadow: 0 0 0 0 randomColor();

  @for $i from 0 through $count {
    $x: #{math.div(random(10000), 10000) * $maxWidth};
    $y: #{math.div(random(10000), 10000) * $maxHeight};
    $shadow: $shadow, #{$x} #{$y} 0 #{random(5)}px randomColor();
  }

  @return $shadow;
}

.item::after {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  width: 1px;
  height: 1px;
  border-radius: 50%;
  box-shadow: shadowSet(100vw, 100vh, 500);
}

.group:nth-child(2) .item {
  animation-delay: -4s;
}

.itemRight {
  transform: rotateY(90deg) translateZ(50px);
}

.itemLeft {
  transform: rotateY(-90deg) translateZ(50px);
}

.itemTop {
  transform: rotateX(90deg) translateZ(50px);
}

.itemBottom {
  transform: rotateX(-90deg) translateZ(50px);
}

.itemMiddle {
  transform: rotateX(180deg) translateZ(50px);
}

.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  grid-area: 1/1/2/2;
  margin: auto !important;
  height: min-content;
  z-index: 1;

  & h1, h2, h3, h4, h5, h6, p {
    color: white;
  }
}
```

</div>

---
layout: two-cols
---
# SlideLayout
components/slide-layout.js

<div style="overflow: scroll;height: 45ch;">

```js
const SlideLayout = ({  
  imagePosition = 'right',
  children,
  direction = '-45deg',
  gradients = '#ee7752, #e73c7e, #23a6d5, #23d5ab'
}) => {
    const imageColumn = (
      <Columns.Column key={0}>
        {children[0]}
      </Columns.Column>
    )
    const contentColumn = (
      <Columns.Column key={1}>
        {children[1]}
      </Columns.Column>
    )
    return (
        <div className={styles.background} 
             style={{backgroundImage: `linear-gradient(${direction}, ${gradients})`}}>
            <Container 
                className={styles.slideLayoutContainer}>
                <Columns
                    className={`${styles.slideLayout} ${imagePosition === 'right' ? styles.columnNormal : styles.columnReverse}`}>
                    { imagePosition === 'right' 
                      ? [contentColumn, imageColumn] 
                      : [imageColumn, contentColumn] }
                </Columns>
            </Container>
        </div>
    )
}
```

</div>

::right::

components/SlideLayout.module.scss

<div style="height: 49ch; overflow: scroll;">

```scss
@use "~bulma/sass/utilities/mixins";

@keyframes backgroundGradient {
  0% {
    background-position: 0 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0 50%;
  }
}

.background {
  height: 100%;
  animation: backgroundGradient 10s ease infinite;
  background-size: 300% 300%;
}

.slideLayout {
  display: flex;
  height: 100%;
  align-items: center;
  margin: 0;
  & :global(.column) {
    display: flex;
    justify-content: center;
  }
}

@include mixins.mobile {
  .slideLayout {
    & :global(.column) {
      width: 100%;
      & article {
        max-width: 80%;
      }
    }

    width: 100%;
    height: min-content;
  }
}

@include mixins.mobile {
  .columnReverse {
    flex-direction: column-reverse;
  }

  .columnNormal {
    flex-direction: column;
  }
}

.slideLayoutContainer {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
```

</div>

---
layout: two-cols
---
# SimpleMessage
components/simple-message.js


<div style="width: 118%;">

```js
import {Message} from "react-bulma-components";
import IconText from "./icon-text";

export default function SimpleMessage({children, icon, title, color}) {
    return (
        <Message color={color}>
            <Message.Header>
                <IconText icon={icon} >{title}</IconText>
            </Message.Header>
            <Message.Body><p>{children}</p></Message.Body>
        </Message>
    );
}
```

# IconText (右)
</div>

::right::

<div style="width:90%;margin-left: 20%;">

# 效果图

<img src="/simple-message.jpg"  />

```js
import React from "react"
import {FontAwesomeIcon} 
from "@fortawesome/react-fontawesome"
const IconText = ({icon, children, 
                   color = null, size = null}) => {
    return (<span className="icon-text">
      <FontAwesomeIcon color={color} 
                       size={size} 
                       className="icon" 
                       icon={icon}/>
      <span>{children}</span>
    </span>)
}
export default IconText
```

</div>
---

# LazyImage
components/lazy-image.js

```js
import {LazyLoadImage} from 'react-lazy-load-image-component';
import {Card} from 'react-bulma-components'
export default function LazyImage({alt, ...props}) {
    return (
        <Card>
            <div className='card-image'>
                <LazyLoadImage
                    alt={alt}
                    effect="blur"
                    threshold={100}
                    {...props}
                />
            </div>
            <Card.Footer>
                <Card.Footer.Item>
                    {alt}
                </Card.Footer.Item>
            </Card.Footer>
        </Card>
    );
}
```

---

# _app.js
pages/_app.js

```js
import '../styles/globals.css'
import {config} from "@fortawesome/fontawesome-svg-core";

config.autoAddCss = false

function MyApp({Component, pageProps}) {
    return <Component {...pageProps} />
}

export default MyApp
```

---
layout: section
---

# 登录/注册 页面

---

# 登录
pages/login.js

<div style="height: 420px;overflow: scroll;">

```js
const getParams = () => new URLSearchParams(window.location.search);
export default function Login() {
    const form = (
        <Formik initialValues={{username: '', password: ''}}
                onSubmit={(values, helpers) => {
                    let formData = new FormData();
                    formData.append('username', values.username);
                    formData.append('password', values.password);
                    let params = getParams();
                    let r = axios.post('/api/user-login', formData)
                    r.then(res => {
                        let redirect_uri = params.get('redirect_uri');
                        if (redirect_uri) {
                            redirect_uri += '?code=' + res.data.access_token + '&state=' + params.get('state');
                            window.location.href = redirect_uri;
                        } else {
                            alert("不知道您想要登录什么应用程序，即将重定向到首页，要访问 JupyterHub, 请从主页访问。");
                            window.location.href = '/';
                        }
                    }).catch(err => {
                        if (err?.response?.data?.detail) {
                            const detail = err.response.data.detail;
                            if (typeof detail == "string") {
                                helpers.setFieldError('username', detail);
                            } else {
                                helpers.setFieldError('username', "请输入用户名和密码！");
                            }
                        } else {
                            helpers.setFieldError('username', '我们这边出现了一些问题，请联系网站管理员');
                        }
                        helpers.setSubmitting(false);
                    });
                }}>
            {({isSubmitting}) => (<Form>
                <FieldContainer>
                    <UserIcon/>
                    <Field type="text" name="username" placeholder="请输入用户名" required/>
                </FieldContainer>
                <StyledErrorMessage name="username" component="div"/>
                <FieldContainer>
                    <PasswordIcon/>
                    <Field type="password" name="password" placeholder="请输入密码" required/>
                </FieldContainer>
                <StyledErrorMessage name="password" component="div"/>
                <StyledButton type="submit" disabled={isSubmitting}>
                    登录
                </StyledButton>
            </Form>)}
        </Formik>
    );
    return (
        <FormLayout form={form}>
            <Head>
                <title>登录</title>
            </Head>
            <StyledLink href="/register">没有帐号？点此注册</StyledLink>
        </FormLayout>
    );
}
```

</div>

---

# 注册页面
pages/register.js

```js
const removeEmptyValues = obj => Object.keys(obj).forEach((k) => obj[k] == null && delete obj[k]);
const validateUsername = username => {
    let error;
    if (username === 'admin') error = '管理员不想让你起这个名字';
    else if (!/^[a-zA-Z0-9_]{4,16}$/.test(username))
        error = "用户名长度应该在4-16位之间，只能由字母、数字或下划线组成";
    return error;
}
const validatePassword = password => {
    let error;
    if (!/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[#?!@$%^&*_=|+-]).{8,128}$/.test(password))
        error = "密码长度应该在8-128位之间，必须包含大写字母、小写字母、数字和#?!@$%^&*_=|+-之中的至少一个特殊字符";
    return error;
}
const validate = values => {
    const errors = {};
    errors.username = validateUsername(values.username);
    errors.pwd__first = validatePassword(values.pwd__first);
    if (values.pwd__first !== values.pwd__second)
        errors.pwd__second = "两次输入的密码不一致";
    removeEmptyValues(errors);
    return errors;
}
```

---

# 注册页面
pages/register.js


<div style="height: 420px;overflow: scroll;">

```js
export default function Register() {
    const form = (
        <Formik 
            initialValues={{username: '', pwd__first: '', pwd__second: '', invite_code: ''}}
            autoComplete={false}
            validate={validate}
            onSubmit={(values, helpers) => {
                let r = axios.post('/api/user-register/', {
                    username: values.username,
                    password: values.pwd__first,
                    invite_code: values.invite_code
                })
                r.then(_ => {
                    alert("注册成功, 即将跳转到登录界面");
                    window.location.href = `${window.location.origin}/login${window.location.search}`;
                }).catch(err => {
                    if (err?.response?.status === 429) {
                        helpers.setFieldError('username', '休息一下吧，您的注册请求过于频繁');
                    } else if (err?.response?.status === 418) {
                        helpers.setFieldError('invite_code', '无效的邀请码，不知道的话就别乱试了，你试不出来的，嘿嘿 (╯°□°）╯︵ ┻━┻');
                    } else if (err?.response?.data?.detail) {
                        const detail = err.response.data.detail;
                        if (typeof detail === 'string')
                            helpers.setFieldError('username', detail);
                    } else {
                        helpers.setFieldError('username', '我们这边出现了一些问题，请联系网站管理员');
                    }
                    helpers.setSubmitting(false);
                });
            }}>
              {({isSubmitting}) => (<Form>
                  <FieldContainer>
                      <UserIcon/>
                      <Field type="text" name="username" placeholder="请输入用户名"
                            autoComplete="off" required/>
                  </FieldContainer>
                  <StyledErrorMessage name="username" component="div"/>
                  <FieldContainer>
                      <PasswordIcon/>
                      <Field type="password" name="pwd__first" placeholder="请输入密码"
                            autoComplete="off" required/>
                  </FieldContainer>
                  <StyledErrorMessage name="pwd__first" component="div"/>
                  <FieldContainer>
                      <PasswordIcon/>
                      <Field type="password" name="pwd__second" placeholder="请再次输入密码" autoComplete="off" required/>
                  </FieldContainer>
                  <StyledErrorMessage name="pwd__second" component="div"/>
                  <FieldContainer>
                      <InviteCodeIcon/>
                      <Field type="text" name="invite_code" placeholder="请输入邀请码" autoComplete="off" required/>
                  </FieldContainer>
                  <StyledErrorMessage name="invite_code" component="div"/>
                  <StyledButton type="submit" disabled={isSubmitting}>
                      注册
                  </StyledButton>
              </Form>)}
        </Formik>
    );

    return (
        <FormLayout form={form}>
            <Head>
                <title>注册</title>
            </Head>
            <StyledLink href="/login">已有帐号？点此登录</StyledLink>
        </FormLayout>
    );
}
```

</div>

---
layout: two-cols
---

# 登录/注册页面的组件

FormLayout, Container 和 Title

```js
const FormLayout = ({form, children}) => (
    <Container>
        <StyledJupyterIcon/>
        <Title/>
        {form}{children}
    </Container>
);
```

<div style="float: left;">

```js
const Container = styled.div`
  margin: 90px auto 0;
  width: 360px;
  background: #dde1e7;
  border-radius: 10px;
  padding: 40px 30px;
  box-shadow: 
    -3px -3px 7px #ffffff73,
    2px 2px 5px 
    rgba(94, 104, 121, 0.288);
`
```

</div>

<div style="float: left; margin-left: 20px;">

```js
const StyledTitle 
  = styled.div`
  font-size: 24px;
  font-weight: 600;
  margin-bottom: 35px;
  color: #000;
`
const Title = () => (
  <StyledTitle>
  数据分析系统统一身份认证
  </StyledTitle>
)
```

</div>

::right::

<img src="/page.png" style="width:300px;margin-left: 40px;"/>

---
layout: two-cols
---

# 登录/注册页面的组件
FieldContainer 和 StyledButton

<div style="width: 600px;">

<div style="float: left;overflow: scroll;height: 410px;">

```js
const FieldContainer = styled.div`
  height: 50px;
  width: 100%;
  display: flex;
  position: relative;
  margin-top: 20px;

  & input {
    height: 100%;
    width: 100%;
    padding-left: 55px;
    font-size: 18px;
    outline: none;
    border: none;
    color: #595959;
    background: #dde1e7;
    border-radius: 25px;
    box-shadow: 
      inset 2px 2px 5px #babecc,
      inset -5px -5px 10px #ffffff73;
  }

  & input:focus ~ label {
    box-shadow: 
      inset 2px 2px 5px #babecc,
      inset -1px -1px 2px #ffffff73;
  }

  & svg {
    position: absolute;
    left: 13px;
    top: 11px;
    width: 28px;
    line-height: 30px;
    color: #595959;
  }
`
```

</div>

<div style="float: left;overflow: scroll;height: 410px;">

```js
const StyledButton = styled.button`
  margin: 25px 0 0 0;
  width: 100%;
  height: 50px;
  color: #000;
  font-size: 18px;
  font-weight: 600;
  background: #dde1e7;
  border: none;
  outline: none;
  cursor: pointer;
  border-radius: 25px;
  box-shadow: 
    2px 2px 5px #babecc,
    -5px -5px 10px #ffffff73;

  &:focus {
    color: #3498db;
    box-shadow: 
      inset 2px 2px 5px #babecc,
      inset -5px -5px 10px #ffffff73;
  }
`
```

</div>

</div>

::right::

<img src="/page.png" style="width:300px;margin-left: 160px;"/>

---
layout: two-cols
---

# 登录/注册页面的组件
StyledLink

<div style="width: 600px;">

<div style="float: left;overflow: scroll;height: 410px;">

```js
const StyledA = styled.a`
  margin: 25px 0 0 0;
  width: 100%;
  display: block;
  height: 50px;
  color: #000;
  font-size: 18px;
  font-weight: 600;
  background: #dde1e7;
  border: none;
  text-align: center;
  outline: none;
  cursor: pointer;
  border-radius: 25px;
  box-shadow: 
     2px 2px 5px #babecc,
    -5px -5px 10px #ffffff73;
  line-height: 50px;

  &:focus {
    color: #3498db;
    box-shadow: 
      inset 2px 2px 5px #babecc,
      inset -5px -5px 10px #ffffff73;
  }
`
```
</div>

<div style="float: left;">

```js
const StyledLink = 
({href, children}) => (
  <StyledA 
  onClick={ 
  () => window.location.href = href 
  + window.location.search
  }>
    {children}
  </StyledA>
)
```

```js
import {ErrorMessage} from "formik";
const StyledErrorMessage = 
styled(ErrorMessage)`
  color: red;
  font-size: 1.1em;
  margin-top: 0.4em;
  margin-bottom: 0.4em;
`
```


</div>

</div>

::right::

<img src="/error.png" style="width:300px;margin-left: 160px;"/>