---
title: 使用git precommit完成前端eslint校验
date: 2017-06-17 09:11:55
tags: [precommit,eslint,"规范"]
reward: true
---

### 使用git precommit完成前端eslint校验

eslint——代码检查工具。可以在一定程度上规范前端的代码风格。也可以在编译前找到一些小错误。一个团队开发。由eslint规范代码风格可以让开发都少踩许多坑
<!-- more -->
### 借助precommit勾子

git在commit的时候会去执行.git/hooks/pre-commit文件,并且pre-commit如果exit非0值git commit就会被停止。我们需要做的是在commit的时候校验我们需求校验的文件。在没有全部通过的情况下返回非0值。

当然也可以在push的时候借助pre-push勾子完成校验（不在此次讨论范围内）

### 具体实现

写bash总感觉总感觉不是前端做的事，所幸的我们可以借助一个模块[pre-commit](https://github.com/observing/pre-commit),我们只需要在 package.json中写上相应字段去运行检查js的文件就好了：

我的配置是：
## package.jons
``` javascript
{
    "pre-commit": {
        "silent": true,
        "run": ["checkJs"]
    },
    "scripts": {
        "checkJs": "node ./bin/lintJs.js",
    }
}

```

## ./bin/lintJs.js
``` javascript
const shell = require("shelljs");
const _ = require("lodash");
const path = require("path")
const { split, isEmpty, filter, last, at, curry, map, groupBy, partialRight, join, get } = _
const beforeTime = "2017/05/22";
const commonEslintrcPath = path.resolve(__dirname, "../.eslintrc.js");
const doExec = cmd => new Promise(resolve => shell.exec(cmd, { silent: true }, (code, stdout, stderr) => resolve({ code, content: stdout || stderr })))
const isScript = filename => /\.jsx?$/.test(filename)
const isFileHasLogbeforeTime = curry(function (time, file) {
    return doExec(`git log --before=${time} -- ${file}`).then(({ content }) => ({
        exist: !isEmpty(content),
        file
    }))
})
const isFilebefore = isFileHasLogbeforeTime(beforeTime);
function doEslint(filenames) {
    const [afterPaths, beforePaths] = map(filenames, name => join(name, " "))
    return Promise.all([
        isEmpty(afterPaths) ? null : doExec(`eslint ${afterPaths}`),
        isEmpty(beforePaths) ? null : doExec(`eslint -c ${commonEslintrcPath} ${beforePaths}`)
    ])
}
function filterFiles(item) {
    const [HEAD] = at(item, [0]);
    return /[ACMR]/.test(HEAD)
}
function gloupyByDate(allFiles) {
    return Promise.all(map(allFiles, isFilebefore))
        .then(partialRight(groupBy, "exist"))
        .then(res => {
            const beforeDate = map(res["true"], "file")
            const afterDate = map(res["false"], "file")
            return isEmpty(afterDate) && isEmpty(beforeDate) ? Promise.reject("files empty") : [afterDate, beforeDate]
        })
}
function getFilesUntranck() { //Add Copied Modified Renamed 
    return new Promise((resolve, reject) => {
        shell.exec("git status -s projects/ydxs_front", { silent: true }, (code, stdout) => {
            const files = _(stdout)
                .split("\n")
                .filter(filterFiles)
                .map(str => last(split(str, " ")))
                .value()
            isEmpty(files) ? reject("empty files list") : resolve(files);
        })
    })
}
getFilesUntranck()
    .then(files => filter(files, isScript))
    .then(gloupyByDate)
    .then(doEslint)
    .then(ress => {
        const [after, before] = ress
        const afterError = get(after, "content")
        const beforeError = get(before, "content")
        beforeError && console.log("旧文件:\n" + beforeError)
        afterError && console.log("新文件:\n" + afterError)
        shell.exit(get(after, "code") || get(before, "code"))
    })
```


具体的逻辑是通过 ```git status -s``` 查看当前文件状态并且根据状态筛选出 ACMR(added copied modified rename)状态的文件(具体含义请自行查看git文档)，再根据后辍名筛选出需要lint的文件——.js或者jsx,接着我们还应用了一套规则老文件与新文件应用不同的eslintrc配置用于平滑过滤。老文件的校验会比较松一些。通过```git log --until 2017/05/22 xx.js```可以查看出xx.js在2017/05/22之前有没有记录。如果没有记录就认为是新文件，如果有记录就认为是老文件。最后通过```eslint```命令校验文件并且在不通过的时候```shell.exit(code)```此时的code应该为非0值以阻止commit动作。 还有一个小技巧，可以指定```git status -s director1 director2```来筛选目录。

最后 ```git commit -m "something" -n```使用```-n```是可以跳过precommit的。我更推荐的是开发者使用编辑器自带的工具来完成代码的校验。纯前端技术是没法强制校验的，只能做为辅助。要强制校验入库的话只能选用服务器端来完成咯。