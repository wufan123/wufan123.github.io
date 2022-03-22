---
title: charles常用技巧
date: 2022-03-22 13:58:40
categories: 
    - [前端]
tags: [charles]
---
 {% asset_img cover.webp  %} 
## Charles修改接口返回数据(Breakpoints)
可以在上方的proxy中添加需要断点的请求
 {% asset_img 1.png %}
也可以直接在请求链接上右键菜单选择Breakpoints。设置完成之后，发起请求，charles会进入断点模式，如下图
 {% asset_img 2.png %} 
这边选择`Edit Request`可以修改请求
 {% asset_img 3.png %} 
点击`Execute`会进去下一步，等待服务器返回，得到返回后，再次进入断点
 {% asset_img 4.png %}
这边选择`Edit Response`可以修改返回
 {% asset_img 5.png %} 






