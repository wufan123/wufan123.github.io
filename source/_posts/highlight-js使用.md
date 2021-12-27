---
title: highlight.js使用
date: 2021-12-23 16:45:46
categories: 
    - [前端]
tags: [javascript]
---

一个web上高亮语法的js，[官网](https://highlightjs.org/)

使用(CDN)：

``` html
<link rel="stylesheet"
      href="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@11.3.1/build/styles/default.min.css">
<script src="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@11.3.1/build/highlight.min.js"></script>
<script>hljs.highlightAll();</script>
```
要更换主题的话，替换css，这边`default.css`是默认主题，要换主题的改css文件名称就可以了，参见[主题列表](https://highlightjs.org/static/demo/)，比如要用`Vs 2015`，改成

``` html
<link rel="stylesheet"
      href="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@11.3.1/build/styles/vs2015.min.css">
``` 