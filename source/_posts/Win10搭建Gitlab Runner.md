---
title: Win10搭建Gitlab Runner
date: 2022-03-22 16:36:52
categories:
  - [CI/CD]
tags: [Gitlab Runner]
---
 {% asset_img cover.webp %}

## CI/CD


**持续集成（CI）**是在源代码变更后自动检测、拉取、构建和（在大多数情况下）进行单元测试的过程。
**持续交付（CD）**通常是指整个流程链（管道），它自动监测源代码变更并通过构建、测试、打包和相关操作运行它们以生成可部署的版本
持续交付包含持续集成（自动检测源代码变更、执行构建过程、运行单元测试以验证变更），持续测试（对代码运行各种测试以保障代码质量），和（可选）持续部署（通过管道发布版本自动提供给用户）。


## gitlab 使用

gitlab 一般是公司搭建的，如果公司还搭建了`Shared runners`那么就可以开始捣鼓`CI/CD`了，没有的话，就可以在自己的服务器或 pc 上搭建了。
在 gitlab 指定项目的`setting`中找到`CI/CD`
 {% asset_img 1.png %}
在页面中展开`runner`选项，可以看到项目的独立 token
 {% asset_img 2.png %}

<!-- more -->   

## 下载 Git Runner


去官网下载`Gitlab Runner`，[Install GitLab Runner on Windows](https://docs.gitlab.com/runner/install/windows.html)，
下载完成后可以重命名一下`gitlab-runner.exe`，主要方便后续操作，找到该目录或者放到一个指定目录
 {% asset_img 3.png %}
点击左上方文件菜单以管理员身份打开`powershell`
 {% asset_img 4.png %}
注册 runner

```Bash
./gitlab-runner.exe register
```

然后出现以下内容
 {% asset_img 5.png %}
第一个是你 gitlab 的地址
第二个是该项目的独立 token，请参考第一部分的说明[gitlab 使用](./#gitlab-使用)
中间的 description，tags，notes 都可以放空，最后的 executor 选 shell，完成之后，在同目录下会生成一个`config.toml`，打开文件编辑
 {% asset_img 6.png %}
将上图的指示的内容改为

```text
shell = "powershell"
```

然后再运行

```Bash
.\gitlab-runner.exe install
.\gitlab-runner.exe start
```

 {% asset_img 8.png %}
此时再去 gitlab 查看项目 setting 中的 runner 时

 {% asset_img 7.png %}
已有可用的 runner，然后在 gitlab 的 CI/CD 面板中可以运行`run pipeline`
 {% asset_img 9.png %}
具体执行什么任务，则是在你项目里面的`.gitlab-ci.yml`中描述
 {% asset_img 10.png %}
