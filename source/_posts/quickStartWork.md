---
title: nodeJs快速启动前端开发环境
date: 2017-05-20 17:23:38
tags:
---
### 每次天上班。都进行着这一系列活动。
* 切到工作目录运行开发环境。 
* 打开编辑器
* 打开浏览器，进入到某页面

### 接着就可以去打水了

今天想着能不能自动化这些操作。于是自己用node来写了一段程序。理论上来说更应该用bash来写的。可惜自己不是很懂bash。 代码如下:

``` javascript
var shell = require("shelljs")
var opn = require("opn")
const { exec, cd } = shell
const workspace = "D:\\git\\yunke_frontend";
cd(workspace)
exec(`code ${workspace}`)//用vscode打开工作 目录的文件
exec(`npm start -- run my dev server`, { async: true })
setTimeout(() => {
    opn('http://theUrlToMyWorkerSpace.com/xx', { app: 'chrome' });
}, 15000) //延时15秒 等编译完成再打开浏览器 

```
主要用到了 [*shelljs*](https://github.com/shelljs/shelljs) 和 [*opn*](https://github.com/sindresorhus/opn)
