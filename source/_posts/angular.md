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

#### 7.4Ng Switch 和Ng SwitchCase
我们可以用在angular模板中使用switch case语句。就像在javascript中使用switch..case一样。

参考代码片段。isMetric是一个组件上定义的变量。如果它的值为真,就会显示Degree Celsicus,否则就会显示Fahrenheit

**注意 ngSwitch是一个属性指令而ngSwitchCase是一个结构指令**
```html
<div [ngSwitch]="isMetric">
  <div *ngSwitchCase="true">Degree Celsius</div>
  <div *ngSwitchCase="false">Fahrenheit</div>
</div>
```
请注意，我们应该使用ngSwitchDefault指令来展示默认元素，以备所有值都不配置时不会什么都不显示。

如果isMetric是布尔值变量，那么下面的代码与之前的代码是等效的。
```html
<div [ngSwitch]="isMetric">
  <div *ngSwitchCase="true">Degree Celsius</div>
  <div *ngSwitchDefault>Fahrenheit</div>
</div>
```

#### 7.5Input装饰器
类内的值是可以配置通过指令获取的，值由使用指令的组件提供。

看看下面的代码片段。这是一个展示登陆字段的组件(组件是指令的一种)。

逻辑组件使用登陆组件，可以设置showRegister为true，就会显示注册按钮。想使用将值提供给类的功能，使用Input装饰器装饰类就好了。

```javascript
Import Input()
import { Component, OnInit, Input } from '@angular/core';
```
*译者注： Import Input()不是合法的语法，此处可以是手误*

装饰器装饰。
```javascript
@input() showRegister: boolean;
```

在组件模板中使用提供的值，示例如下。

```html
<div>
  <input type="text" placeholder="User Id" />
  <input type="password" placeholder="Password" />  
  <span *ngIf="showRegister"><button>Register</button></span>
  <button>Go</button>
</div>
```

当使用这个组件的时候，像这样提供值给它：

```html
<login shouldShowRegister="true"></login>
```
使用绑定数据代替直接提供值，使用如下语法：
```html
<app-login [shouldShowRegister]="isRegisterVisible"></app-login>
```

*译者注： app-login应该是上文的login*

我们可以提供另一个绑定属性的名字心区分变量的名字，看看下面的例子:

```javascript
@Input("should-show-register") showRegister: boolean;
```

现在，使用`should-show-register`代替`showRegister`:

```html
<app-login should-show-register="true"></app-login>
```

#### 7.6输出装饰器

事件被指令发给使用指令的组件(或指令)，在那个登陆的例子中，点击登陆，一个事件应该携带着user id 和password被发出。

看看下面代码，导入Output装饰器和EventEmitter，

```javascript
import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';
```
申明一个类型为EventEmitter的Output事件
```javascript
@Output() onLogin: EventEmitter<{userId: string, password: string}>;
```
注意onLogin是一个EventEmitter的泛型类型，它期望被发出时带着含有user id和password的对象。

接着初始化对象
```javascript
constructor() {
  this.onLogin = new EventEmitter();
}
```

组件会发出事件，在例子中，当用户点击登陆时，它会通过接下来的函数发出一个事件。

看例子：

```html
<button (click)="loginClicked(userId.value, password.value)">Go</button>
```

具体的事件处理函数：
```javascript
loginClicked(userId, password) {
  this.onLogin.next({userId, password})
}
```
当使用这个组件的时候，只需要提供登陆函数函数响应onLogin输出事件就行了。
```html
<app-login (onLogin)="loginHandler($event)"></app-login>
```

登陆句柄能从组件接收用户id和密码
```javascript
loginHandle(event) {
  console.log(event);
  // Perform login action.
}
// Output: Object {userId: "sampleUser", password: "samplePassword"}
```
与输入装饰器相似的是，输出装饰器可以使用另一个事件名字暴露给使用自己的组件/指令。它可以不使用变量名。看看以下代码段。
```javascript
@Output("login") onLogin: EventEmitter<{userId: string, password: string}>;
```
像这样使用组件：
```html
<app-login (login)="loginHandler($event)"></app-login>
```

### 08 脏检查策略(change detection strategy)
Angular自上向下传播改变，从父组件传递给子组件。

Angular中的每一个组件都有一个等价的脏检查类。一旦组件的数据模型更新了，它就会对比新值和旧值，数据模型的改变会被应用到DOM上。

对于组件上的输入类型为number和string型的，它们的值是不可变的。一旦值改变了，脏检查类就会标记改变，DOM就会更新。

对于对象类型，比较是否变化的方法可能是以下的一种：

**Shallow Comparison**: 浅比较对比对象的的引用，如果一个或者多个对象中的字段更新了，而对象的应用没有更新的话，浅比较会认为没有值改变发生。这种方法与不可变对象类型完美组合。意思是，对象不能被修改，改变对象的唯一方法是创建一个新的对象并将引用指向新对象，那样脏检查会被完美触发。

**Deep Comparison**: 深比较迭代对象中的每一个字段，并且与之前的值对比。这种策略在可变对象中也可以生效。也就是说一旦某个字段改变了。脏检查就能检查出来。

#### 8.1组件的脏检查策略
在一个组件中，我们可以使用一个或两个检查策略来装饰它。

#### ChangeDetectionStrategy.Default：
像它的名字一样，它是默认的策略。当没有明确指定其它策略时。它就生效。

使用这个策略，当组件接收一个对象作为输入时，深比较会在每一次改变时进行。也就是说，当有一个字段改变的时候，也会迭代所有的字段，然后标记出改变了的那一个。接着改变DOM。

看看下面的示例:

```javascript
import { Component, OnInit, Input, ChangeDetectionStrategy } from '@angular/core';
 
@Component({
  selector: 'app-child',
  template: '  <h2>{{values.title}}</h2> <h2>{{values.description}}</h2>',
  styleUrls: ['./child.component.css'],
  changeDetection: ChangeDetectionStrategy.Default
})
export class ChildComponent implements OnInit {
 
// Input is an object type.
  @Input() values: {
    title: string;
    description: string;
  }
 
  constructor() { }
 
  ngOnInit() {}
 
}
```
注意到检查策略明确的提到了是*ChangeDetectionStrategy.Default*，*Input*申明了一个对象中包含了两个字段，并且描述了类型。

这是父组件使用的模块片段。
```html
<app-child [values]="values"></app-child>
```
父组件中的事件处理句柄如下。这个句柄可能在用户的行为或者其它事件触发。
```javascript
updateValues(param1, param2){
  this.values.title = param1;
  this.values.description = param2;
}
```
注意，我们更新了值，但是没有改变对象的引用。但是子组件更新了DOM因为脏查检策略是深比较的。

#### ChangeDetectionStrategy.onPush
默认的策略在定位改变上很有效，但是由于比较时需要循环整个对象来定位改变太消耗性能。我们有性能更优的脏检查策略，OnPush策略。它只是浅对比输入对象和之前对象的引用。

我们刚才看到的例子在OnPush策略下是不会生效的。在例子中，我们只是改变了对象中的值却没有改变对象的引用，子组件无法监控到改变，以至于无法更新DOM。

当我们将组件脏检查策略改为OnPush时，输入的对象就需要是不可变的,我们应该在每一次改变时，为输入对象创建一个新的实例。这要对外引用改变了，每一次改变都会因此正确定位到。

看看下面的代码
```javascript
updateValues(param1, param2){
  this.values = { // create a new object for each change.
    title: param1,
    description: param2
  }
}
```

**注意**: 上面的*updateValues*事件处理函数在OnPush策略下是正确的。更明智的做法是使用[immutable.js](https://facebook.github.io/immutable-js/)——强制不可变数据类型的实现。或者是使用可观测对象RxJS或者其它的可观测库。

// TODO http://www.dotnetcurry.com/angular/1385/angular-4-cheat-sheet
