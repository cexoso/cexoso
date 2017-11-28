---
title: angular速查表
date: 2017-11-27 16:56:55
tags: angular
---

[本文翻译自此处](http://www.dotnetcurry.com/angular/1385/angular-4-cheat-sheet)，略有改动。

**abstract**：这份angular速查表是为了你快速的进行angular开发的，教程使用typescript和angular v4.

-------

Angular是一个应用于移动端和web应用的Javascript框架。作为客户端的Javascript框架，它能更好的写出运行在浏览器中的前端应用。今天，Angular的进步让现代网页开发技术能够创建运行在桌面浏览器的web应用，移动端网页，各式各样的移动端，以及移动端的本地应用。

Angular应用可以使用ES5,ECMAScript 2015,或者Typescript编码。因为Typesciprt的开源，写Angular应用Typescript是一个流行的选择。并且Typescript拥有强大的类型检测能力，和诸多其它特性，提供自动完成，导航和重构，这些在有应用中非常有价值的特性。由于Typescript利用ES6作为其核心基础的一部分，你会感觉像打正像使用ES6一样上瘾。

这篇文章是一份用Typesciprt写Angular应用的速查表，它是[V Keerti Kotaru ](http://www.dotnetcurry.com/author/v-keerti-kotaru)完成的，带领你快速进行Angular开发的指引。

## Angular 4 速查表

### 01.Components （组件）
组件是搭建Angular应用的积木。一个组件是HTML模板，Typescript（或者Javascript）Class（类)构成的。在Angular中写一个类并且使用组件装饰器装饰它，就简单的创建了一个模板。

参考以下示例：

```javascript
import { Component } from '@angular/core';
@Component({
  selector: 'hello-ng-world',
  template: `<h1>Hello Angular world</h1>`
})
export class HelloWorld { 
}
```

Component类从angular核心包中导入，组件装饰器允许接收一系列的元数据用来描述组件。元数据有很多的字段，此处将列出一些重要的：

**selector**: (选择器)是用来标识组件的名字，在上面的例子中，`hello-ng-world`就是用来关联当前组件的选择器，它可以被用在其它HTML代码或者模板中。

**template**： (模板)是组件的标记。它被处理后，可以用来绑定组件类中的数据，样式，和逻辑。

**templateUrl**：(模板Url)是一个外部模板文件的Url,为了更好的代码组织，我们通常会把模板写到另一个文件中并且用templateUrl来指定这个文件的路径。

**styles**: （样式)是用来定义组件样式的。样式的作用域只对组件有效果。

**styleUrls**: (样式路径)是用来指明描述组件样式的css文件路径的。我们可以在数组中指明一个更多个css文件。类和其它的样式将被应用在组件模板中。

### 02.Data-binding (数据绑定)
angualr应用中使用字符串插值是最简单的数据展示手段了。看看接下来在angualr模板中展示变量值的语法。

`` `<h1>Hello {{title}} world</h1>` ``

title是一个变量名，注意包裹字符串的单引号(重音符`)，它是es6中将数据和字符串混合的语法。

完整的组件代码片段如下:
```javascript
import { Component } from '@angular/core';
@Component({
  selector: 'hello-ng-world',
  template: `<h1>Hello {{title}} world</h1>`
})
export class HelloWorld { 
title = 'Angular 4'; 
}
```

想要在文本字段中展示值，使用中括号包裹DOM属性“*value*“，就像使用DOM属性一样简单自然。每一个Angular的初学者都乐于接受。”title“是一个在组件类中定义的变量名。

`<input type="text" [value]="title" />`

这种绑定可以用在任何HTML属性中，比如接下来的例子，title用在文本字段中的placeholder属性。

`<input type="text" [placeholder]="title" />`

###03.Events
为了给DOM事件绑定上组件中的函数，我们使用如下语法。将DOM事件包裹在小括号中，就好像它调用了组件中的函数一样。

`<button (click)="updateTime()">Update Time</button>`

#### 3.1 接受用户输入
当用户敲击键盘或者在下拉菜单中、单选按钮中做选择的时候。组件中的变量值需要被更新。

-----
我们可以使用*events*来达到目的。

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

在上面接收用户输入的例子中，*$event*被赤裸裸的暴露在组件函数，它表征了最初被触发的完整的DOM事件。不管怎么说，函数所需要的仅仅是一个用于文本字段的值而已。

接下来的示例，使用文本字段上的模板引用值，优雅地实现了同样的功能。

`<input type="text" #label1 (change)="updateValue(label1.value)"/>`

注意绑定在change上的*updateValue*函数,它接收了label1(模板的引入值)上的value值。更新函数可以简单的设置值到类内变量中了。
```javascript
updateValue(value: any) {
    this.label = value;
}
```

#### 3.3 盒子中的香蕉
至Angular2以上的版本、双向数据绑定已经不在内置功能中了。接下来的代码展示了，当用户改变文本字段中的值的时候，它并不会直接更新h1标签中title的值。

```html
<h1> {{title}} </h1>
<input type="text" [value]="title">
```

使用如下的双向绑定语法，它是混合了值绑定和事件绑定的简易格式 - [{}]。这被乐观地称之为盒子里的香蕉。*（译者表示真的不知道这段话到底有什么幽默点）*

```html
<h1> {{title}} </h1>
<input type="text" [(ngModel)]="title" name="title">
```

#### 04.ngModel 和表单字段
ngModel不单单在双向数据绑定上有用。值得肯定的是它还会附加CSS样式类，用来表明表单字段的控制。

看看这一段表单字段 - 姓和名。ngModel被设置成了视图模型上的fname和lname字段(vm变量在组件内定义了)。

```html
<input type="text" [(ngModel)]="vm.fname" name="firstName" #fname required /> 
{{fname.className}} <br />
<input type="text" [(ngModel)]="vm.lname" name="lastName" #lname /> 
{{lname.className}}
```
请注意，如果ngModel在form标签内使用的话，它就需要搭配一个name属性。控制也会同时注册到form上(上层使用name属性值，如果不提供name属性会产生一个如下的错误)。

> If ngModel is used within a form tag, either the name attribute must be set or the form control must be defined as 'standalone' in ngModelOptions.

*(意思为: 如果ngModel在form标签内使用的话，要不就提供name属性，要不就在ngModelOptions国的standalong中定义表单控制)*

正确的使用会在用户使用表单输入的时候添加如下的CSS类。

**ng-untouched** - 表单一加载完成后字段被会被设置。用来表示用户还没有使用这个字段和聚集它

**ng-touched** - 当用户通过键盘或者鼠标聚集字段，又移动到其它字段上后。这个类会被添加。

**ng-pristine** - 这个类用来表明字段上的值一直没有改变过。

**ng-dirty** - 这个类在用户改变字段值的时候会被设置


//TODO http://www.dotnetcurry.com/angular/1385/angular-4-cheat-sheet