---
title: nodejs中的package.json文件
date: 2021-12-27 16:36:19
categories: 
    - [nodejs]
tags: []
---

关于package.json文件的[官方文档说明](https://docs.npmjs.com/cli/v8/configuring-npm/package-json/),英文好的的同学可以自行前往查看，下面是对一些文件中的常见字段的说明：
## name 
项目名称，如果要发布的话，这个字段必须
## version
版本，同上

<!-- more -->

## scripts
定义了脚本命令，比如

``` json
"scripts": { 
     "prepare": "git submodule init && git submodule update && git submodule foreach git pull origin master"
  }
```

上面定义prepare脚本命令，通过`npm run prepare`执行`git submodule init && git submodule update && git submodule foreach git pull origin master`

## dependencies
项目运行依赖的模块

## devDependencies
项目开发所需要的模块

## bin
可执行文件，会安装到系统的`PATH`中，如果是全局安装，直接使用包名即可，确保bin中的js文件是以`#!/usr/bin/env node`开头，不然会无法识别javascript语句

## main 
main字段指定了加载入口文件，`require()`加载这个文件


## config 
添加命令行的环境变量，比如

``` json
{
    "config":{"port":8080}
}
```
在脚本就可以使用

``` JS
process.env.npm_package_config_port
```



