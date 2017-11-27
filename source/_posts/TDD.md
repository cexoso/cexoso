---
title: 敏捷开发TDD
date: 2017-05-13 11:03:20
tags: tdd
reward: true
---
这篇文章主要是安利测试驱动开发的


TDD（测试驱动开发）是敏捷开发中的一项核心实践和技术。区别于传统的前端开发TDD的有如下优势：

1.专注
---
编程是需要反馈的，你写下一段代码的时候，心中就**期望**着这段代码的行为。在前端开发的时候，我们可能这样：
````
a. 编写代码，
b. 打开浏览器进入挂载了代码的网页
c. 触发代码运行（比如必需在填写好一个表单并点击保存后，才可能触发数据处理的函数）
d. 通过看输出或者页面效果来评价这段代码是不是正常的动作了
````
在以上的步骤中，可能在a和b之前还有代码的编译过程（像webpack把es6，typescript编译合并的过程）。
<!-- more -->

像这种开发某个功能点的流程，时间大部分都浪费在了刷新浏览器（热更新，自动刷新可能解决）上、触发代码运行上、环境的搭建上（让node跑起前端环境打开浏览器，输入网页这些行为都算是搭建开发环境）。重要的是如果不能一次通过的话，你还要重复一次等待编译，刷新浏览器，触发逻辑并再次观察结果的行为，这时你可能会在心理面骂自己我怎么会烦这种低级的错误。

现在，有了TDD。你只需要简单的在控制台上敲击命令启动TDD环境并指明你要跑的文件（可能是单个，也可能是多个如：*.spec.js,、\_\_test\_\_/**/*）接着在测试文件中写上你期望去测试的点。接着你就可以去写逻辑代码了。并且你的每次编写都会等到迅速的反映。你只需要一边看着控制台给出的结果一边编码就可以了。

2.重构或再次开发
---
试想一个场景。你的代码通过了测试人员的测试上线了。但是在某种情况下，可能是因为数据不符合当初的约定、也可能是用户的操作偏离了你的想像，出现了不预期的后果。而你不得不要求测试人员帮你在测试环境重现客户的场景。也可能更糟糕你得代理到正式环境下，用正式的环境去调试你的代码。（不能更糟了）

而如果之前使用TDD开发的话。此时你只需要运行那个出了问题的文件的测试用例。并添加上一条测试用例。并且重构代码或者加入新的代码。由于之前的用例约束，一般不会由于本次修改而导致以前的bug复现。

3.亲近程序员
---
我相信对于程序员来说去理解业务是比较痛苦的，去深入理解业务更痛苦。TDD就是用程序的语言来描述的。看别人的代码的时候，别人想做什么是一目了然的。

4.强制代码规范
---
这一点也可以说是缺点。用了TDD，你写代码就再也不可以那么随意了。
关于怎么写出可测试的代码。请看下一篇。