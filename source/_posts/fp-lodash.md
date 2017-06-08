---
title: 使用lodash进行编程
date: 2017-05-21 20:19:02
tags: lodash
reward: true
---
# Pointfree编程风格（草稿）
![timo](fp-lodash/timo.png)
<!-- more -->
如下示例
```javascript
import { someAction } from "./action"
function mapStateToProps(state) {
    return state.path.to.my.state;
}
function mapActionToProps(dispatch) {
    return bindActionCreators({
        goDetail: someAction
    },dispatch)
}
connect(mapStateToProps, mapActionToProps)
```
可以用lodash像如下实现
```javascript
import { someAction } from "./action"
import { property,partial } from "lodash"
const mapStateToProps = property("path.to.my.state")
const mapActionToProps = partial(bindActionCreators,{
    goDetail: someAction
})
connect(mapStateToProps, mapActionToProps)
```

以下示例出自[reselect](https://github.com/reactjs/reselect)
```javascript
import { createSelector } from 'reselect'

const shopItemsSelector = state => state.shop.items
const taxPercentSelector = state => state.shop.taxPercent

const subtotalSelector = createSelector(
  shopItemsSelector,
  items => items.reduce((acc, item) => acc + item.value, 0)
)
```
改写后
```javascript
import { createSelector } from 'reselect'
import { property,reduce,partialRight } from "lodash"
const shopItemsSelector = property("shop.items")
const taxPercentSelector = property("shop.taxPercent")

const subtotalSelector = createSelector(
  shopItemsSelector,
  partialRight(reduce,(acc, item) => acc + item.value,0)  
)
```