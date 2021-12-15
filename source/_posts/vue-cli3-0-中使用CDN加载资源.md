---
title: vue-cli3.0+中使用CDN加载资源
date: 2021-12-10 16:04:46
categories: 
    - [前端]
tags: [vue]
---
在`vue.config.js`中配置使用CDN加载的modules（使用全局变量代替），比如
``` js
 chainWebpack: config => {
    config.set('externals', {
        'vue': 'Vue',//"库名:引入后的别名" 
        'vue-router': 'VueRouter',
        'vuex': 'Vuex',
        'mint-ui': 'MintUI'
    })
  }
```
然后在public中的`index.html`加入链接,比如上面所列的库

``` html
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vue-router@3.5.3/dist/vue-router.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vuex@3.6.2/dist/vuex.min.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/mint-ui/lib/style.css" />
    <script src="https://unpkg.com/mint-ui/lib/index.js"></script> 
```

 资源要自己找，这边推荐两个CDN平台
- [jsdelivr](https://www.jsdelivr.com/) 
- [unpkg](https://www.unpkg.com/)

这样就配置完成了，改成CDN的话，有些库的使用方式需要改一下，参照每个库的官方说明即可，比如[Vue Router](https://router.vuejs.org/zh/)
说明了CDN和NPM安装的区别:

- CDN

``` HTML
<!-- 在 Vue 后面加载 vue-router，它会自动安装的： -->
<script src="/path/to/vue.js"></script>
<script src="/path/to/vue-router.js"></script>

```
- NPM

``` bash
npm install vue-router
```

如果在一个模块化工程中使用它，必须要通过 Vue.use() 明确地安装路由功能：

``` js
import Vue from 'vue'
import VueRouter from 'vue-router'
Vue.use(VueRouter)
```

其他的不一一说明了


