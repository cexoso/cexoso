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

### 03.Events
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

**ng-valid** - 这个类会在表单字段校验通过，不失败时添加。例如当一个required字段有值。

**ng-invalid** - 这个类会在表单字段失败是添加，例如当一个required字段没有值。

这个特性允许特定场景中自定义CSS类。例如一个无效的字段会高亮红色的背景来提示用户。

为组件中的元素定义如下的CSS。

```css
.ng-invalid {
  background: red;
}
```

### 05.ngModule

创建一个Angular模块只需要在类上使用ngModule装饰就可以了。

一个模块帮助打包一系列的Anglar制品如:组件(component)，指令(directives)，管道(pipes)等。它有助于（ahead-of-time）AoT编译。一个模块向其它的模块导出组件指令和管道。它也可能依赖其它模块。

参考下列例子。

*注意*： 所有的组件都必须在模块中申明

```javascript
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { MyAppComponent }  from './app.component';
 
@NgModule({
  imports:      [ BrowserModule ],
  declarations: [ MyAppComponent ],
  bootstrap:    [ MyAppComponent ]
})
export class MyAppModule { }
```

从angular核心包中导入ngModule。它是一个装饰函数，可以被应用于一个类上。我们可以提供以下元数据来创建一个模块。

**imports**: 依赖的*BrowserModule*使用imports指定。注意imports支持数组。多个依赖可以被同时申明。

**declarations**: 所有与当前模块关联的组件，都应用使用declarations数组申明。

**bootsctrap**: 每一个Angular应用需要有一个根模块。根模块需要一个根组件用于在index.html中渲染。在这个例子中。MyAppComponent (通过相对路径导入)就是第一个被加载的根组件。

### 06.Service
在Angular中创建服务，是为了代码片的复用。比如用来接受服务器端API的服务，或者是提供配置信息的服务等。

创建一个服务很简单，只需要从@angular/core中导入@injectable并装饰一个类。

这允许angular创建一个实例并且向组件或者其它服务注入它。

看看这个假设的例子。它创建一个叫TimeService的类用来提供一个固定格式的时间值。这个服务是可以跨应用来使用的。getTime()函数可以在应用的组件中被调用。

```javascript
import { Injectable } from '@angular/core';
 
@Injectable()
export class TimeService {
  constructor() { }
  getTime(){
    return `${new Date().getHours()} : ${new Date().getMinutes()} : ${new Date().getSeconds()}`;
  }
}
```

#### 6.1依赖注入(DI)

在介绍服务寻一段中，我们看到了使用Injectable()装饰器创建一个服务。Injectable帮助Angular注入一个由服务类创建的实例。它仅当服务有其它依赖是才被需要。无论如何，一个好的实践方式是为所有的服务类都加上Injectable()装饰(即使某些类并不存在依赖)来保证代码同一性。

有了依赖注入，Angular可以为组件创建服务的实例。这为对象的创建单例的实现都提供了便利，也让代码更益于单元测试。

#### 6.2提供一个服务

将服务注入到组件中，需要将服务在providers数组中申明。这个操作可以在模块级别(能大范围访问)或者组件级别(只在组件域中访问)完成。在这个点服务会被Angular实例化。

这里有一个组件例子，被注入的服务实例在组件和子组件中都存在。

```javascript
import { Component } from '@angular/core';
import { TimeService } from './time.service';
@Component({
  selector: 'app-root',
  providers: [TimeService],
  template: `<div>Date: {{timeValue}}</div>,
})
export class SampleComponent {
  // Component definition
}
```

为了完成依赖注入的过程，要在构造函数中用参数的形式指服务。

```javascript
export class SampleComponent {
  timeValue: string = this.time.getTime(); // Template shows
  constructor(private time: TimeService){
  }
}
```

上文提到的TimeService具体实现如下，

```javascript
import { Injectable } from '@angular/core';
 
@Injectable()
export class TimeService {
  constructor() { }
  getTime(){
    let dateObj: Date = new Date();
    return `${dateObj.getDay()}/${dateObj.getMonth()}/${dateObj.getFullYear()}`;
  }
}
```

#### 6.3提供模块级别的服务

当TimeService在模块级被提供时(相对于组件级),服务的实例就在多模块(和应用)中可见了。它就像是应用中的单例一样。

```javascript
@NgModule({
  declarations: [
    AppComponent,
    SampleComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [TimeService],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

#### 6.4 使用替换语法来提供服务
在上面的例子中，provider是下面语法的简写。

```javascript
// instead of providers: [TimeService] you may use the following,
  providers: [{provide: TimeService, useClass: TimeService}]
```

它允许专门的类或者替代类的实现(例子中的TimeService)。新的类可以是派生类，或者相对于原始类，拥有相似签名的函数(鸭子类型)。

`providers: [{provide: TimeService, useClass: AlternateTimeService}]`

在下面的这个例子中，AlternateTimeService提供日期和时间，而原始的TimeService只提供日期。

```javascript
@Injectable()
export class AlternateTimeService extends TimeService {
 
  constructor() {
    super();
   }
 
  getTime(){
    let dateObj: Date = new Date();
    return `${dateObj.getDay()}/${dateObj.getMonth()}/${dateObj.getFullYear()}
                   ${dateObj.getHours()}:${dateObj.getMinutes()}`;
  }
}
```

**注意**下面语句的使用方式，Angular会注入一个新的服务实例。

`[{provide: TimeService, useClass: AlternateTimeService}]
`

若要使用已经存在的实例，使用useExisting来代替useClass

`providers: [AlternateTimeService, {provide: TimeService, useExisting: AlternateTimeService}]`

#### 6.5提供一个接口和它的实现

一个好的想法是提供一个接口并且实现这个接口。参考下面的接口。

```javascript
interface Time{
    getTime(): string
}
export default Time;
```

它可以被TimeService实现如下。
```javascript
@Injectable()
export class TimeService implements Time {
  constructor() { }
 
  getTime(){
    let dateObj: Date = new Date();
    return `${dateObj.getDay()}/${dateObj.getMonth()}/${dateObj.getFullYear()}`;
  }
}
```
这一点上，我们不可以用实现Time接口和TimeService类的方法完成之前的例子。

就如同下面的代码并不能够正常动作，因为在Typescript中，接口是不会编译成等价的javascript的。

`providers: [{provide: Time, useClass: TimeService}] // Wrong`

为了解决这个问题，使用从@angular/core中导入的Inject(装饰器)和InjectionToken。
```javascript
// create an object of InjectionToken that confines to interface Time
let Time_Service = new InjectionToken<Time>('Time_Service');
 
// Provide the injector token with interface implementation.
providers: [{provide: Time_Service, useClass: TimeService}]
  
// inject the token with @inject decorator
constructor(@Inject(Time_Service) ts,) {
      this.timeService = ts;
}
 
// We can now use this.timeService.getTime().
```

#### 6.6提供一个值

我们并不需要总是写一个类来当作服务。

这一段语法像我们展示了使用JSON对象来作为服务。

```javascript
providers: [{provide: TimeService, useValue: {
  getTime: () => `${dateObj.getDay()} - ${dateObj.getMonth()} - ${dateObj.getFullYear()}`
}}]
```
**注意**: 有一个相似的实现可以代替`6.5提供一个接口和它的实现`节中，使用useValue代替useClass。其余的实现保持不变。

```javascript
providers: [provide: Time_Service, useValue: {
  getTime: () => 'A date value'
}}]
```

### 0.7指令
从Angular2以上，指令被宽泛的分成如下几类:

1. **组件** - 包含模板。它们是最初级的Angular应用的积木。相关参考本文的组件部分
2. **结构指令** - 用来控制布局的指令，通常用在元素的属性上和控制DOM流。例如NgFor,NgIf等。
3. **属性指令** - 动态地为元素添加行为和样式，例如NgStyle。

#### 7.1Ng-if...else
一个按条件展示模块的结构指令。如下示例。ngIf指令守卫展示一半字符串Time:[with no value]当title为空时。else展示一个模板当没有值赋给title时。

注意这的语法，指令前有一个星号。
```html
<div *ngIf="title; else noTitle">
  Time: {{title}}
</div>
 
<ng-template #noTitle> Click on the button to see time. </ng-template>
```

一旦title赋值为value后，Time:[value]就会被展示，#noTitle模块就会隐藏起来，else也不会被执行。

#### 7.2Ng-Template

Ng-Template是一个结构指令，默认不会展示。它是用来包裹元素内内容的。默认情况下它会将模块渲染成html中的注释。

下而的内容会随便条件展示。

```html
<div *ngIf="isTrue; then tmplWhenTrue else tmplWhenFalse"></div>
<ng-template #tmplWhenTrue >I show-up when isTrue is true. </ng-template>
<ng-template #tmplWhenFalse > I show-up when isTrue is false </ng-template>
```

**注意**: 这种方式可以代换ng-if...else的实现，并且在代码可读性上更好，我们可以用if...then...else。这个例子中元素在条件为真性展示，并且展示的内容移到了模板中维护。

#### 7.4Ng-Container

Ng-Container是另一个用来包裹HTML元素的指令/组件。

我们用div或者span包裹元素。无论如何，在一挺多的应用中，会存在一个默认的样式被应用到元素上。为了行为更可预测，优先推荐Ng-Container。它会包裹元素，却不会渲染成HTML标签。

看看下面的例子。

```html
// Consider value of title is Angular
Welcome <div *ngIf="title">to <i>the</i> {{title}} world.</div>
```
它会渲染成
```
Welcome
to the Angualr world
```
使用Ng-Container代替div，如下代码

```html
Welcome <ng-container *ngIf="title">to <i>the</i> {{title}} world.</ng-container>
```
结果如下
```
Welcome to the Angular world
```
区别在于第一个例子多渲染了一个div而Ng-Container不会渲染成标签。

TODO// http://www.dotnetcurry.com/angular/1385/angular-4-cheat-sheet
