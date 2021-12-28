---
title: vue-cli5源码学习
date: 2021-12-21 16:01:53
categories:
  - [前端]
tags: [vue,cli] 
---
## vue.js

首先查看`packages/@vue/cli/bin/vue.js`文件

<!-- more -->

```javascript
/* @vue/cli-shared-utils集成了一些通用工具，比如chalk(终端字符串样式)，
semver(检查npm包版本)，也封装了一些常用方法，像退出程序，合法判断等 */
const { chalk, semver } = require("@vue/cli-shared-utils");
const requiredVersion = require("../package.json").engines.node;

/* 用Levenshtein distance算法来计算两个字符的差异，这个算法我也没去了解，
这边在后面用来比较错误命令与现有命令，给出建议 */
const leven = require("leven");

function checkNodeVersion(wanted, id) {
  if (!semver.satisfies(process.version, wanted, { includePrerelease: true })) {
    console.log(
      chalk.red(
        "You are using Node " +
          process.version +
          ", but this version of " +
          id +
          " requires Node " +
          wanted +
          ".\nPlease upgrade your Node version."
      )
    );
    process.exit(1);
  }
}

//检查npm版本
checkNodeVersion(requiredVersion, "@vue/cli");

const fs = require("fs");
const path = require("path");
//将 Windows 反斜杠路径转换为斜杠路径：foo\\bar➔foo/bar
const slash = require("slash");
//解析参数选项
const minimist = require("minimist");
// enter debug mode when creating test repo
if (
  slash(process.cwd()).indexOf("/packages/test") > 0 &&
  (fs.existsSync(path.resolve(process.cwd(), "../@vue")) ||
    fs.existsSync(path.resolve(process.cwd(), "../../@vue")))
) {
  process.env.VUE_CLI_DEBUG = true;
}

//commander是nodejs命令行的解决方案
const program = require("commander");
const loadCommand = require("../lib/util/loadCommand");
/*
.
.
. 
关注下create命令，其他命令先略过
*/
program
  .command('create <app-name>')
  .description('create a new project powered by vue-cli-service')
  .option('-p, --preset <presetName>', 'Skip prompts and use saved or remote preset')
  .option('-d, --default', 'Skip prompts and use default preset')
  .option('-i, --inlinePreset <json>', 'Skip prompts and use inline JSON string as preset')
  .option('-m, --packageManager <command>', 'Use specified npm client when installing dependencies')
  .option('-r, --registry <url>', 'Use specified npm registry when installing dependencies (only for npm)')
  .option('-g, --git [message]', 'Force git initialization with initial commit message')
  .option('-n, --no-git', 'Skip git initialization')
  .option('-f, --force', 'Overwrite target directory if it exists')
  .option('--merge', 'Merge target directory if it exists')
  .option('-c, --clone', 'Use git clone when fetching remote preset')
  .option('-x, --proxy <proxyUrl>', 'Use specified proxy when creating project')
  .option('-b, --bare', 'Scaffold project without beginner instructions')
  .option('--skipGetStarted', 'Skip displaying "Get started" instructions')
  .action((name, options) => {
    if (minimist(process.argv.slice(3))._.length > 1) {
      console.log(chalk.yellow('\n Info: You provided more than one argument. The first one will be used as the app\'s name, the rest are ignored.'))
    }
    // --git makes commander to default git to true
    if (process.argv.includes('-g') || process.argv.includes('--git')) {
      options.forceGit = true
    }
    //引入lib中的create
    require('../lib/create')(name, options)
  })

```
<!-- more -->

## create.js
然后我们去看`packages/@vue/cli/lib/create.js`

``` javascript
//fs-extra包含了fs没有提供的方法，并支持promise
const fs = require('fs-extra')
const path = require('path')
//inquirer  提供命令行用户交互
const inquirer = require('inquirer')
const Creator = require('./Creator')
const { clearConsole } = require('./util/clearConsole')
//提示可配置的modules
const { getPromptModules } = require('./util/createTools')
//stopSpinner封装了ora(一个终端的加载提示)
const { chalk, error, stopSpinner, exit } = require('@vue/cli-shared-utils')
//验证项目名称合法
const validateProjectName = require('validate-npm-package-name')

async function create (projectName, options) {
  if (options.proxy) {
    process.env.HTTP_PROXY = options.proxy
  }
  //process.cwd() 当前工作目录
  const cwd = options.cwd || process.cwd()
  const inCurrent = projectName === '.'
  // path.relative() 算出当前文件夹名
  const name = inCurrent ? path.relative('../', cwd) : projectName
  // 目标文件夹的绝对路径
  const targetDir = path.resolve(cwd, projectName || '.')
  // 验证工程名是否合法，返回{ validForNewPackages: true, validForOldPackages: true }
  const result = validateProjectName(name)
  if (!result.validForNewPackages) {
    console.error(chalk.red(`Invalid project name: "${name}"`))
    result.errors && result.errors.forEach(err => {
      console.error(chalk.red.dim('Error: ' + err))
    })
    result.warnings && result.warnings.forEach(warn => {
      console.error(chalk.red.dim('Warning: ' + warn))
    })
    exit(1)
  }
  //目标文件夹已存在的一些处理
  if (fs.existsSync(targetDir) && !options.merge) {
    if (options.force) {
      await fs.remove(targetDir)
    } else {
      await clearConsole()
      if (inCurrent) {
        const { ok } = await inquirer.prompt([
          {
            name: 'ok',
            type: 'confirm',
            message: `Generate project in current directory?`
          }
        ])
        if (!ok) {
          return
        }
      } else {
        const { action } = await inquirer.prompt([
          {
            name: 'action',
            type: 'list',
            message: `Target directory ${chalk.cyan(targetDir)} already exists. Pick an action:`,
            choices: [
              { name: 'Overwrite', value: 'overwrite' },
              { name: 'Merge', value: 'merge' },
              { name: 'Cancel', value: false }
            ]
          }
        ])
        if (!action) {
          return
        } else if (action === 'overwrite') {
          console.log(`\nRemoving ${chalk.cyan(targetDir)}...`)
          await fs.remove(targetDir)
        }
      }
    }
  }

  //具体工作交给创造器
  const creator = new Creator(name, targetDir, getPromptModules())
  await creator.create(options)
}

module.exports = (...args) => {
  return create(...args).catch(err => {
    stopSpinner(false) // do not persist
    error(err)
    if (!process.env.VUE_CLI_TEST) {
      process.exit(1)
    }
  })
}

```

然后我们去看`packages/@vue/cli/lib/creator.js`

``` js
const path = require('path')
//debug一个小型调试器
const debug = require('debug')
//上面说过的  inquirer提供命令行用户交互
const inquirer = require('inquirer')
//events 事件触发器，下面主要基于事件触发和响应的
const EventEmitter = require('events')
const Generator = require('./Generator')
//lodash提供一些常用工具方法，像深复制，防抖什么的
const cloneDeep = require('lodash.clonedeep')
//sortObject 排序对象属性，默认unicode，或者按照指定顺序来排序,这边用来排序plugin
const sortObject = require('./util/sortObject')
//getVersions 用来获取最新的cli plugin版本,大致实现,比较缓存时间,如果超过一天就去下载远程package,保证获取较新版本
const getVersions = require('./util/getVersions')
//PackageManager 包的管理更新
const PackageManager = require('./util/ProjectPackageManager')
//clearConsole 生成一个标题再通过换行清空cli输出
const { clearConsole } = require('./util/clearConsole')
const PromptModuleAPI = require('./PromptModuleAPI')
const writeFileTree = require('./util/writeFileTree')
const { formatFeatures } = require('./util/features')
const loadLocalPreset = require('./util/loadLocalPreset')
const loadRemotePreset = require('./util/loadRemotePreset')
const generateReadme = require('./util/generateReadme')
const { resolvePkg, isOfficialPlugin } = require('@vue/cli-shared-utils')
/*  
...省略部分
*/
module.exports = class Creator extends EventEmitter {
   //...
  async create (cliOptions = {}, preset = null) {
    //待学习...
  }

}
```
