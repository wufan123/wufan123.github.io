---
title: npx
date: 2021-12-28 10:30:06
categories: 
    - [nodejs]
tags: [npx]
---
npm 从5.2版开始，增加了 npx 命令，主要解决的问题是调用项目内部安装的模块，比如
```
npm install webpack -D
```
要调用webpack的话只能在项目脚本或者package.json的scripts字段，如果要在cli调用，要这样

``` bash
#根目录下
$ node-modules/.bin/webpack -v
```
使用npx的话可以直接调用

``` bash
npx webpack -v
```

原理是npx运行时会到`node_modules/.bin`和`$PATH`里检查命令是否存在，所以系统命令也可以调用，其他具体参见[官方说明](https://www.npmjs.com/package/npx) 