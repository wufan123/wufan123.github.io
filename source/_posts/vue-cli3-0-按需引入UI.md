---
title: vue-cli3.0+按需引入mint-ui
date: 2021-12-10 15:54:45
categories: 
    - [前端]
tags: [vue]
---
### 安装
首先需要`babel-plugin-component`
``` bash
npm install babel-plugin-component -D 
```
### 配置
然后在`babel.config.js`中配置
``` bash
module.exports = {
  presets: ["@vue/cli-plugin-babel/preset"],
  plugins: [
       [
      "component",
      {
        libraryName: "mint-ui",
        style: true,
      }
    ],
  ]
};
```
### 使用
在`main.js中`使用
``` js
import { Swipe, SwipeItem } from "mint-ui";
```


