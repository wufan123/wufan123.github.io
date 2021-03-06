---
title: webpack和gulp的区别
date: 2021-12-21 11:33:59
categories: 
    - [前端]
tags: [webpack,gulp] 
---
Gulp 的定位 __Task Runner__, 就是用来跑一个一个任务的。

放在以前比如我想用sass写css, coffee写js, 我必须手动的用相应的compiler去编译各自的文件，然后各自minify。这时候designer给你了两张新图片，好嘞，接着用自己的小工具手动去压缩图片。
后来前端人不能忍了，搞出个自动化这个流程的 Grunt/Gulp, 比如你写完代码后要想发布production版本，用一句`gulp build`就可以
+ rm 掉 dist文件夹中以前的旧文件
+ 自动把sass编译成css, coffee编译成js
+ 压缩各自的文件，压缩图片，生成图片sprite
+ 拷贝minified/uglified 文件到 dist 文件夹

但是它没发解决的是 js module 的问题，是你写代码时候如何组织代码结构的问题.
<!-- more -->
之前大家可以用 require.js, sea.js 来 require dependency, 后来出了一个 webpack 说 我们能不能把所有的文件(css, image, js) 都用 js 来 生成依赖，最后生成一个bundle呢？ 所以webpack 也叫做 __file bundler__ .

同时 webpack 为了解决可以 require 不同文件的需求引入了loader, 比如面对sass文件有

+ sass-loader, 把sass 转换成 css

+ css-loader, 让 webpack 能识别处理 css

+ style-loader, 把识别后的 css 插入到 html style中
类似的识别es6 有babel-loader

本来这就是 webpack 的初衷，require everything, bundle everything. 一开始 webpack 刚出来的时候大家都是把它结合着 gulp 一起用的， gulp 里面有个 gulp-webpack，就是让 webpack 专门去做module dependency的事情, 生成一个bundle.js文件，然后再用 gulp 去做一些其他杂七杂八minify, uglify的事情。 后来人们发现 webpack 有个plugins的选项， 可以用来进一步处理经过loader 生成的bundle.js，于是有人写了对应的插件， 所以minify/uglify, 生成hash的工作也可以转移到webpack本身了，挤掉了gulp这部分的市场份额。 再后来大家有发现 npm/package.json 里面的scripts 原来好好用啊，调用任务的时候就直接写一个简单的命令，因为 gulp 也不就是各种插件命令的组合呀，大部分情况下越来越不需要 gulp/grunt 之类的了 ref. 所以你现在看到的很多新项目都是package.json里面scripts 写了一堆，外部只需要一个webpack就够了。

转自[drbelfast在segmentfault的部分回答](https://segmentfault.com/q/1010000008058766)，侵删
