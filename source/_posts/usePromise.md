---
title: 使用promise实现Maybe
date: 2017-06-21 21:08:23
tags: [fp,函数编程,promise]
reward: true
---

# 使用promise实现Maybe

假如一个代码
```javascript
function doSomething(list) {
    if (Array.isArray(list)) {
        const head = list[0]
        if (head !== undefined) {
            const {username,tel} = head;
            return `${username}'s tel is ${tel}`
        } else {
            throw new Error("should has a Element")
        }
    } else {
        throw new Error("list must be a Array")
    }
}
try {
    const ret = doSomething([{username: "xiaomin",tel: "13333332223"}])
    console.log(ret)
} catch (e) {
    console.error(e)
}
```
<<<<<<< HEAD
<!-- more -->
=======
>>>>>>> 0650605332a4a75917ad0f0c97e6ad6bee0787df
为了保证代码的撸捧性，做了好多判断。让人看着非常的头痛。有时候我们做业务，只关心正确的代码怎么走。但是往往还要写一堆异常的代码情况。而这些异常的情况我们又不是很care的时候。就感觉很难受了。

## Maybe、Either
```javascript
function doSomething(list) {
    return Promise.resolve(list)
        .then(list=>Array.isArray(list) ? list[0] : Promise.reject("list must be a Array"))// then1
        .then(head=>head !== undefined ? head : Promise.reject("should has a Element")) //then2
        .then(({username,tel})=>`${username}'s tel is ${tel}`) //then3
}
doSomething([{username: "xiaomin",tel: "13333332223"}])
    .then(console.error,console.log)
```

好像没有感觉好多少？该判断的一个都没有少。使用promise来做这些事，有几个好处。
* 第一个就是函数```doSomething```总会返回promise值（```either```）。来表明结果要不就出错，要不就是正确的(有点废话)。而之前的写法是```doSomething```要不会返回一个正确的值，要不就抛出一个错误。

* 第二个是流程从if的嵌套中解脱出来了。then的方法更像是顺序执行的。这种写法对异步更友好。假如有一天需要根据username来从服务器获取tel，我们也是只需要加一个then就可以了。

* 使用then强迫拆分，实现上是让代码更解耦的。因为每一步是```lambda```，而用if else的写法则把代码揉在了一起。

## 函数编程库 lodash
其实lodash里面就有考虑过这种情况

lodash版本的代码
```javascript
import {chain,flowRight,head} from "lodash"
function doSomething(list) {
    return chain(list).map(({username,tel})=>`${username}'s tel is ${tel}`).head().value();
}
```
lodash 这种做法是Maybe型的，不会告诉你怎么错的。它帮你默认了错误时的效果，比如 ```doSomething([])```会返回一个undefined。在实际中我感觉lodash的这种做法很实用。这也是为什么用lodash可以减少很多代码的原因。