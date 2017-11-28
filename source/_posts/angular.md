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

Component类从angular核心包中导入，组件装饰器允许接收一系列的元数据用来描述组件。元数据有很多的字段，此处将列出一些重要的：

**selector**: (选择器)是用来标识组件的名字，在上面的例子中，`hello-ng-world`就是用来关联当前组件的选择器，它可以被用在其它HTML代码或者模板中。

**template**： (模板)是组件的标记。它被处理后，可以用来绑定组件类中的数据，样式，和逻辑。

**templateUrl**：(模板Url)是一个外部模板文件的Url,为了更好的代码组织，我们通常会把模板写到另一个文件中并且用templateUrl来指定这个文件的路径。

**styles**: （样式)是用来定义组件样式的。样式的作用域只对组件有效果。

**styleUrls**: (样式路径)是用来指明描述组件样式的css文件路径的。我们可以在数组中指明一个更多个css文件。类和其它的样式将被应用在组件模板中。

### 02.Data-binding (数据绑定)
angualr应用中使用字符串插值是最简单的数据展示手段了。看看接下来在angualr模板中展示变量值的语法。

`` `<h1>Hello {{title}} world</h1>` ``

title是一个变量名，注意包裹字符串的单引号(重音符`)，它是es6中将数据和字符串混合的语法。

完整的组件代码片段如下:
```javascript
import { Component } from '@angular/core';
@Component({
  selector: 'hello-ng-world',
  template: `<h1>Hello {{title}} world</h1>`
})
export class HelloWorld { 
title = 'Angular 4'; 
}
```

想要在文本字段中展示值，使用中括号包裹DOM属性“*value*“，就像使用DOM属性一样简单自然。每一个Angular的初学者都乐于接受。”title“是一个在组件类中定义的变量名。

`<input type="text" [value]="title" />`

这种绑定可以用在任何HTML属性中，比如接下来的例子，title用在文本字段中的placeholder属性。

`<input type="text" [placeholder]="title" />`

###03.Events
为了给DOM事件绑定上组件中的函数，我们使用如下语法。将DOM事件包裹在小括号中，就好像它调用了组件中的函数一样。

`<button (click)="updateTime()">Update Time</button>`

#### 3.1 接受用户输入
当用户敲击键盘或者在下拉菜单中、单选按钮中做选择的时候。组件中的变量值需要被更新。

-----
我们可以使用*events*来达到目的。

接下来的片段更新了文本字段的值。

```html
<!— Change event triggers function updateValue on the component -->
<input type="text" (change)="updateValue($event)" />
 
  updateValue(event: Event){
    // event.target.value has the value on the text field.
    // It is set to the label.
    this.label = event.target.value;
  }

```

#### 3.2 更好的接收用户的输入

在上面接收用户输入的例子中，*$event*被赤裸裸的暴露在组件函数，它表征了最初被触发的完整的DOM事件。不管怎么说，函数所需要的仅仅是一个用于文本字段的值而已。

接下来的示例，使用文本字段上的模板引用值，优雅地实现了同样的功能。

`<input type="text" #label1 (change)="updateValue(label1.value)"/>`

注意绑定在change上的*updateValue*函数,它接收了label1(模板的引入值)上的value值。更新函数可以简单的设置值到类内变量中了。
```javascript
updateValue(value: any) {
    this.label = value;
}
```

#### 3.3 盒子中的香蕉
至Angular2以上的版本、双向数据绑定已经不在内置功能中了。接下来的代码展示了，当用户改变文本字段中的值的时候，它并不会直接更新h1标签中title的值。

```html
<h1> {{title}} </h1>
<input type="text" [value]="title">
```