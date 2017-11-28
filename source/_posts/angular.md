---
title: angular速查表
date: 2017-11-27 16:56:55
tags: angular
---

[本文翻译自此处](http://www.dotnetcurry.com/angular/1385/angular-4-cheat-sheet)，略有改动。

**abstract**：这份angular速查表是为了你快速的进行angular开发的，教程使用typescript和angular v4.

-------

Angular是一个应用于移动端和web应用的Javascript框架。作为客户端的Javascript框架，它能更好的写出运行在浏览器中的前端应用。今天，Angular的进步让现代网页开发技术能够创建运行在桌面浏览器的web应用，移动端网页，各式各样的移动端，以及移动端的本地应用。

Angular应用可以使用ES5,ECMAScript 2015,或者Typescript编码。因为Typesciprt的开源，写Angular应用Typescript是一个流行的选择。并且Typescript拥有强大的类型检测能力，和诸多其它特性，提供自动完成，导航和重构，这些在有应用中非常有价值的特性。由于Typescript利用ES6作为其核心基础的一部分，你会感觉像打正像使用ES6一样上瘾。

这篇文章是一份用Typesciprt写Angular应用的速查表，它是[V Keerti Kotaru ](http://www.dotnetcurry.com/author/v-keerti-kotaru)完成的，带领你快速进行Angular开发的指引。

## Angular 4 速查表

### 01.Components （组件）
组件是搭建Angular应用的积木。一个组件是HTML模板，Typescript（或者Javascript）Class（类)构成的。在Angular中写一个类并且使用组件装饰器装饰它，就简单的创建了一个模板。

参考以下示例：

```javascript
import { Component } from '@angular/core';
@Component({
  selector: 'hello-ng-world',
  template: `<h1>Hello Angular world</h1>`
})
export class HelloWorld { 
}
```