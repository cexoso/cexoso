---
title: 为什么我们可以用函数编程
date: 2017-08-26 10:05:39
tags:
---

# 表达式

一个常见的表达式：`sql`，例如：
```sql
SELECT * FROM student;
```
从`student`表中获取所有的数据。

```sql
CREATE TABLE Persons
(
    PersonID int,
    LastName varchar(255),
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255)
);
```
创建一张表。

`表达式（expression）`表达的是要什么，而不是怎么去做。
<!-- more -->
## 一些简单的`javascrip`示例
```javascript
// 示例1 求和
const nums = [1,2,3,4,5]
const total = 0;
for (let i = 0; i < nums.length; i++) {
    total += nums[i]
}
// 示例2 获取所有及格学生
const allStudent = [{score: 59, name: "xiaowang"},{score: 60, name: "xiaohong"}];
const passStudent = []
for (let i = 0; i< allStudent.length; i++) {
    const student = allStudent[i]
    if (student.score > 60) {
        passStudent.push(student)
    }
}
```
表达式形式
```javascript
// 示例1 求和
const nums = [1,2,3,4,5]
const total = sum(nums)
// 示例2 获取所有及格的学生
const allStudent = [{score: 59, name: "xiaowang"},{score: 60, name: "xiaohong"}];
const passStudent = filter(allStudent, ({score}) => score > 60)
//或者 
const passStudent = filterAllPass(allStudent)
```
示例中并没有说`sum`和`filter`或者`filterAllPass`是哪里来的。这里只是想表达一下表达式的简洁。然而简单的封装一下
```javascript
const filterAllPass = (allStudent) => {
    const passStudent = []
    for (let i = 0; i< allStudent.length; i++) {
        const student = allStudent[i]
        if (student.score > 60) {
            passStudent.push(student)
        }
    }
    return passStudent;
}
```
就可以把指令式的封装成了函数式的表达式，难道函数式是指令式的多此一举吗?

# 组合

函数式绝不是指令式的多此一举。如果真的需要`filterAllPass`函数。函数式编程也会有自己的一套解决方法`组合`.简单的说`filterAllPass`就是`遍历`(这里是`filter`不是`map`)、`取属性`、`判断大小`组合起来的。如果
* 遍历 -> filter
* 取属性 -> props
* 判断大小 -> gt
* 组合 -> .

那么 `const filterAllPass = filter(compose(gt(60), props("score")))` 

遗憾的是javascript不可以自定义操作符，也没有`.`操作符来代替`compose`，所以javascript的代码看起来总是会比较长。

`haskell`版本:`filterAllPass = filter $ (> 60) . (props "score")`

# 兼容性

## monad
实现了`map`函数的数据结构就是`monad`。例如`Number.of(1).map(+2).map(*3)` => `Number.of(9)`,此时的`Number`就是一个`monad`。不严格的说。`Promise`就是monad。Promise提供了`then`函数返回新的Promise，就像Monad提供map函数返回新的Monad一样。

## Maybe
`Maybe`是一个Monad，它能在有值(`Just`)的时候进行map没值(`Nothing`)的时候啥都不干。例如：`Maybe.of(just 3).map(+3).map(*3)=>Just 18`。`Maybe.of(Nothing).map(+3).map(*3)=>Nothing`。

## 一些优秀的`js`库
如果你有考虑过函数`const filterAllPass = filter(compose(gt(60), props("score")))`在接收异常的参数的时候会怎么样时，已经有一些优秀的库考虑了。
比如`lodash`的`filter`，在你调用的时候无论如何都会返回一个数组给你。即使你用undefined做为参数得到的也是数组。还有`ramda`的`props`函数，总是会返回你想取的属性的值，不管这个属性存在不存在。也不管你想操作的对象是不是undefined。这种特性很像Maybe的特性，有值就会处理，没有值就不去处理。反正不会因为边界情况而中断。

# 实际项目中的一些例子及lodash、ramda的简介
## 素材1
```javascript
function sumProductAmount2() {
    return function (products) {
        var amount = 0;
        $.each(products, function (i, d) {
            if (d.__type == 'delete') {
                return;
            }
            amount = myMath.add(amount, d.amount);
        });
        return amount;
    };
}
```
`sumProductAmount2`函数内使用了`each`做为函数处理数组数据而不是使用`for`循环来处理。但是在整个实现上，还是延用了指令编程的思想。指明了一个数据要先判断是否是`delete`的数据,再去与迭代`amount`求和

### lodash版本
```javascript
function sumProductAmount2() {
    return function (products) {
        return chain(products)
            .reject(["__type", "delete"])
            .map("amount")
            .reduce((acc, x) => myMath.add(acc, x), 0)
            .value()
    };
}
```
`lodash`版本使用`chain`将`products`打包，使用申明型语句表明了去掉`__type`属性等于`delete`的数据，再将数据根据属性`amount`印射，然后对所有的数据进行迭代`reduce((acc, x) => myMath.add(acc, x.amount), 0)`。最后取值`value()`,整体上已经很好了。这样的函数完全可以写成一个`lamda`表达式(如果不怕难看的话):
```javascript
const sumProductAmount2 = () => products => chain(products)
    .reject(["__type", "delete"])
    .map("amount")
    .reduce((acc, x) => myMath.add(acc, x), 0)
    .value()
```
lodash版本有一些美中不足的是reduce函数虽然已经很fp风格了。但是在语义上并没有尽善尽美。如果有一个函数`.sumWith(myMath.add)`用以代替`.reduce((acc, x) => myMath.add(acc, x), 0)`的话。易读性就会变得非常的好。

### ramda版本
```javascript
import { reduce, map, prop, reject, propEq, compose } from 'ramda'
const sumProductAmount2 = () => compose(
        reduce(myMath.add，0), 
        map(prop("amount")), 
        reject(propEq("__type", "delete"))
)
```
`ramda`库没有语法糖，没有魔法银弹。看看`lodash`版本的`reject`。
```javascript
var users = [
  { 'user': 'barney', 'age': 36, 'active': false },
  { 'user': 'fred',   'age': 40, 'active': true }
];
 
_.reject(users, function(o) { return !o.active; });
// => objects for ['fred']
 
// The `_.matches` iteratee shorthand.
_.reject(users, { 'age': 40, 'active': true });
// => objects for ['barney']
 
// The `_.matchesProperty` iteratee shorthand.
_.reject(users, ['active', false]);
// => objects for ['fred']
 
// The `_.property` iteratee shorthand.
_.reject(users, 'active');
// => objects for ['barney']
```
这个怪异的签名意思是
1. 可以传递一个函数，当返回值为true时reject此数据。
2. 可以传递一个对象，会使用`_.matche`函数调用些对象返回一个函数，之后情况参照示例1.
3. 可以传递一个元组(其实就是长度为2的数组)，会使用`_.matchesProperty`调用返回一个函数，之后参数示例1.
4. 传递一个字符串，应用`_.property`

换言之：
* `_.reject(users, function(o) { return !o.active; });`这种写法是最初始提供的。
* `_.reject(users, { 'age': 40, 'active': true });`写法是`_.reject(users, _matches({ 'age': 40, 'active': true }));`的语法糖。
* `_.reject(users, ['active', false]);`是`_.reject(users, _.matchesProperty('active', false));`的语法糖
* `_.reject(users, 'active');`是`_.reject(users, _.property('active'));`的语法糖

并且这种调用形式在lodash里面被广泛使用。熟悉这种语法糖会感觉非常的方便。而不熟悉的人会感觉代码怪异难懂。

`ramda`函数库并没有类型的语法糖。要想表达属性`__type`等于`delete`必须乖乖使用`propEq("__type", "delete")`，没有这种形式`["__type", "delete"]`

下面介绍一下`ramda`与`lodash`的不同。ramda更容易写出`pointfree`的代码(也许`lodash/fp`也提供了类似方法)。

在示例中 函数`sumProductAmount2`只是为了返回一个函数对数据源`products`做处理。
ramda版本的函数并没有提及这个数据源`products`，它只是表明了一个函数该做的事。
```javascript 
compose(
    reduce(myMath.add，0), 
    map(prop("amount")), 
    reject(propEq("__type", "delete"))
)
```
使用组合的方式组合三个匿名函数。（注意顺序是从右到左从下到上）
1. `reject(propEq("__type", "delete"))`过滤掉了所有`__type`属性为`delete`数据
2. `map`将数据属性`amount`提取出来。
3. `reduce`将所有的数据通过`myMath.add`救和，初始值设置为0。

这三点在介绍`lodash`版本的时候说过了。这里再说一次的意图是`ramda`和`lodash`处理的思想是一致的。只是在写法上不同(`lodash/fp`也许也能这么写，所以`ramda`与`lodash`也许只有`api`设计的不同)


还有一个点`reduce(myMath.add，0)`,由于我知道`myMath.add`中没有使用`this`并且知道`reduce`会为迭代函数提供两个参数`acc`,`value`。我才敢这样做。为了更安全可以写成`reduce((acc,value) => myMath.add(acc,value)，0)`。这种提及了数据的写法很丑。但是让人感觉到安全，因为看到了数据的流向。所以在函数编程里面，尽量不要使用`this`。因为this本身就是为面向对象设计，`this`会让函数不纯。

看不到数据的流向会让人调试很麻烦。除了使用`tdd`开发方式外。还可以通过帮助函数进行调试。
```javascript
const helper = data => {
    console.log(data); //break here；
    return data;
}
compose(
    reduce(myMath.add，0), 
    helper,
    map(prop("amount")), 
    reject(propEq("__type", "delete"))
)
```
`helper`不会影响到正常的流程，只是会提供了一个看见数据的机会。

之前我们提到过`reduce(myMath.add，0)`形式没有`sumWith`更语义化，在`lodash`版本里面我们没法改，但是`ramda`版本我们是可以替换的。

```javascript
const sumWith = fn => reduce(fn，0);

compose(
    sumWith(myMath.add), 
    map(prop("amount")), 
    reject(propEq("__type", "delete"))
)
```
# don't repeat yourself *（dry）*
```javascript
// myMath.add是一个避免浮点数加法精度问题的加法函数函数
// $ 是 jquery(众所周知)
filter('countSMS2', function () {
    return function (products) {
        var num = 0;
        $.each(products, function (i, d) {
            if (d.__type == 'delete') {
                return;
            }
            num = myMath.add(num, d.authorization_type == 1 ? d.authorization_numeric : 0);
        });
        return num;
    };
})
filter('countProject2', function () {
    return function (products) {
        var num = 0;
        $.each(products, function (i, d) {
            if (d.__type == 'delete') {
                return;
            }
            num = myMath.add(num, d.authorization_type == 0 ? d.authorization_numeric : 0);
        });
        return num;
    };
})
```
上文定义了两个过滤器。代码几乎一模一样，一个统计authorization_type 为 1的 一个统计authorization_type 为 0的。

上述代码违背了`dry`，
> 原作者内心：我没有重复编码，第二段是我`CV`的。

先用`lodash`重构一下:
```javascript
filter('countSMS2', function () {
    return function (products) {
        return chain(products)
            .reject(["__type", "delete"])
            .map(({ authorization_type: at, authorization_numeric: an }) => +at === 1 ? an : 0)
            .reduce((acc, x) => myMath.add(acc, x), 0)
            .value()
    };
})
filter('countProject2', function () {
    return function (products) {
        return chain(products)
            .reject(["__type", "delete"])
            .map(({ authorization_type: at, authorization_numeric: an }) => +at === 0 ? an : 0)
            .reduce((acc, x) => myMath.add(acc, x), 0)
            .value()
    };
})
```
接着把公共的部分提出去
```javascript
const sumByType = type => products => chain(products)
    .reject(["__type", "delete"])
    .map(({ authorization_type: at, authorization_numeric: an }) => +at === type ? an : 0)
    .reduce((acc, x) => myMath.add(acc, x), 0)
    .value()
filter('countSMS2', () => sumByType(0));
filter('countProject2', () => sumByType(1))
```
全`lamda`函数写，看起来就很爽

这里用到的就是`curry`化。`sumByType`函数是`curried function`。部分调用后产生偏函数(`partial`)。解决了重复的问题。当然这个sumByType也叫高阶函数(`HOF`)。

# 一点总结
`javascript`在函数式上只是稍具意味，属于东拼西凑，需要搭配第三方库来配合使用。使用`javascript`时应该综合考虑。不应盲目使用编程范式。但是相信之后函数式编程范式在`javascipt`上会得到增强。