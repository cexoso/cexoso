---
title: 前端使用vscode的一些小建议
date: 2017-07-14 11:15:12
tags: [vscode]
---

# 使用vscode的一些小建议

## 安装

- [安装地址](https://code.visualstudio.com/) 作为一个程序员剩下的肯定难不到你了
- [github](https://github.com/Microsoft/vscode) vscode是一个开源软件 如果你遇到麻烦的话你是可以
    * [看issues](https://github.com/Microsoft/vscode/issues)
    * 提rp
    * [看wiki](https://github.com/Microsoft/vscode/wiki)
    * 上面三点基本都用不到所以你点个star就好了

<!-- more -->
## code命令

在window环境下vscode的安装包会帮你在path里添加自己的命令，使用bash敲入`code`可以快速的使用vscode打开文件或者工程

        code . #打开当前目录的所有文件
        code index.html #打开index.html

## 插件推荐

现在vscode的插件推荐已经很完善了。自己去淘也是可以找到好插件的。在这里推荐一些我觉得好的。

* `auto-Open Markdown Preview` 自动打开markdown预览。写作与浏览的时候都挺有用的。
* `ESLint` 可以在界面上看到ESLint的提示。
* `Git Blame` 查看**单行**上次提交者
* `Git History(gitlog)` 查看git提交记录的
* `One Monokai Theme` 一个主题。看起来很舒服。安装后要在设置里面设置才能使用主题
* `Path Autocomplete` 快速输入路径
* `Project Manager` 项目管理，多个项目切换起来很方便

关于git插件，vscode自带的工具可以满足普通需求了。更高端的需求，vscode、插件都没有提供。推荐自己使用git bash命令行。这种命令行能解决的东西，一定要在命令行里面多用，才能有提高。推荐一篇[`git文章`](http://blog.jobbole.com/96088/)

## 设置
可以通过`ctrl + Comma`(Comma是逗号`,`)打开设置界面。也可以通过`文件`->`首选项`->`设置 `打开。

设置很看懂。下面也推荐一些设置。

```json
{
    "workbench.iconTheme": "vscode-icons",
    "terminal.external.windowsExec": "D:\\program files\\Git\\bin\\bash.exe",
    "terminal.integrated.shell.windows": "D:\\program files\\Git\\bin\\bash.exe",
    "editor.fontFamily": "Microsoft YaHei Mono",
    "workbench.colorTheme": "One Monokai",
}
```
分别是 使用vscode-icons系列，外部bash，集成bash，字体(必需你的系统有),主题,One Monokai(之前插件里面安装的主题)。集成命令行与外部命令行设置成自己喜欢的,对我来说太重要了。

## debug
这一点我自己用着都挺迷。平时也不是很用得着。所以就不说了。