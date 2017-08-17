---
title: 下一篇fp介绍的素材
date: 2017-08-17 09:57:16
tags:
---

# 素材1
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
`sumProductAmount2`函数内使用了`each`做为函数处理数组数据而不是使用`for`循环来处理。但是在整个实现上，还是延用了指令编程的思想。指明了一个数据要先判断是否是`delete`的数据,再去与迭代`amount`求和

# lodash版本
```javascript
function sumProductAmount2() {
    return function (products) {
        return chain(products)
            .reject(["__type", "delete"])
            .map("amount")
            .reduce((acc, x) => myMath.add(acc, x), 0)
            .value()
    };
}
```
`lodash`版本使用`chain`将`products`打包，使用申明型语句表明了去掉`__type`属性等于`delete`的数据，再将数据根据属性`amount`印射，然后对所有的数据进行迭代`reduce((acc, x) => myMath.add(acc, x.amount), 0)`。最后取值`value()`,整体上已经很好了。这样的函数完全可以写成一个`lamda`表达式(如果不怕难看的话):
```javascript
const sumProductAmount2 = () => products => chain(products)
    .reject(["__type", "delete"])
    .map("amount")
    .reduce((acc, x) => myMath.add(acc, x), 0)
    .value()
```
lodash版本有一些美中不足的是reduce函数虽然已经很fp风格了。但是在语义上并没有尽善尽美。如果有一个函数`.sumWith(myMath.add)`用以代替`.reduce((acc, x) => myMath.add(acc, x), 0)`的话。易读性就会变得非常的好。

# ramda版本
```javascript
import { reduce, map, prop, reject, propEq, compose } from 'ramda'
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
3. 可以传递一个元组(其实就是长度为2的数组)，会使用`_.matchesProperty`调用返回一个函数，之后参数示例1.
4. 传递一个字符串，应用`_.property`

换言之：
* `_.reject(users, function(o) { return !o.active; });`这种写法是最被提供的。
* `_.reject(users, { 'age': 40, 'active': true });`写法是`_.reject(users, _matches({ 'age': 40, 'active': true }));`的语法糖。
* `_.reject(users, ['active', false]);`是`_.reject(users, _.matchesProperty('active', false));`的语法糖
* `_.reject(users, 'active');`是`_.reject(users, _.property('active'));`的语法糖

并且这种调用形式在lodash里面被广泛使用。熟悉这种语法糖的小朋友会感觉非常的甜。而不熟悉的人会感觉代码怪异难懂。

`ramda`函数库并没有类型的语法糖。要想表达属性`__type`等于`delete`必须乖乖使用`propEq("__type", "delete")`，没有这种形式`["__type", "delete"]`

下面介绍一下`ramda`与`lodash`的不同。ramda更容易写出`pointfree`的代码(也许`lodash/fp`也提供了类似方法)。

在示例中 函数`sumProductAmount2`只是为了返回一个函数对数据源`products`做处理。
ramda版本的函数并没有提及这个数据源`products`，它只是表明了一个函数该做的事。
```javascript 
compose(
    reduce(myMath.add，0), 
    map(prop("amount")), 
    reject(propEq("__type", "delete"))
)
```
使用组合的方式组合三个匿名函数。（注意顺序是从右到左从下到上）
1. `reject(propEq("__type", "delete"))`过滤掉了所有`__type`属性为`delete`数据
2. `map`将数据属性`amount`提取出来。
3. `reduce`将所有的数据通过`myMath.add`救和，初始值设置为0。

这三点在介绍`lodash`版本的时候说过了。这里再说一次的意图是`ramda`和`lodash`处理的思想是一置的。只是在写法上不同(`lodash/fp`也许也能这么写，所以`ramda`与`lodash`也许只有`api`设计的不同)


还有一个点`reduce(myMath.add，0)`,由于我知道`myMath.add`中没有使用`this`并且知道`reduce`会为迭代函数提供两个参数`acc`,`value`。我才敢这样做。为了更安全可以写成`reduce((acc,value) => myMath.add(acc,value)，0)`。这种提及了数据的写法很丑。但是让人感觉到安全，因为看到了数据的流向。所以在函数编程里面，尽量不要使用`this`。因为this本身就是为面向对象设计，`this`会让函数不纯。

看不到数据的流向会让人调试很麻烦。除了使用`tdd`开发方式外。还可以通过帮助函数进行调试。
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
`helper`不会影响到正常的流程，只是会提供了一个看见数据的机会。

之前我们提到过`reduce(myMath.add，0)`形式没有`sumWith`更语义化，在`lodash`版本里面我们没法改，但是`ramda`版本我们是可以替换的。

```javascript
const sumWith = fn => reduce(fn，0);

compose(
    sumWith(myMath.add), 
    map(prop("amount")), 
    reject(propEq("__type", "delete"))
)
```
