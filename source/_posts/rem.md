---
layout: 实现
title: rem
date: 2018-06-26 20:48:36
tags: rem
---

## 淘宝、网易移动端 px 转换 rem 原理，Vue-cli 实现 px 转换 rem

拿iPhone5（640px）的设计稿举例，淘宝的思想是无论当前页面多宽，让10rem = 页面宽度 = 100%，所以1rem = 64px 然后通过dpr设置缩放整个页面，以达到高保真的效果。iPhone的dpr是2，所以设置  <meta name="viewport" content="initial-scale=0.5, maximum-scale=0.5, minimum-scale=0.5, user-scalable=no"> 就可以了适配iPhone5了,当然这个都是有 `lib-flexible`帮我们做了

**具体使用如下:**

1. 在模板index.html中引入lib-flexible
```html
<script src="lib-flexible.js"></script>
```
 or
```html
import 'lib-flexible'
```
2. 在webpack的cssLoaders配置中加入如下配置
```javascript
  const px2remLoader = {
    loader: 'px2rem-loader',
    options: {
      remUnit: 64 //设计稿宽度/10
    }
  // 将 cssLoaders 方法内的generateLoaders的方法内的 loaders 变量添加 px2remLoader 
  const loaders = options.usePostCSS ? [cssLoader, postcssLoader, px2remLoader, lessLoader ] : [cssLoader, px2remLoader, lessLoader ]
```

