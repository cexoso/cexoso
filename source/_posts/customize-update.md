---
title: 自定义react更新机制
date: 2017-06-18 19:23:20
tags: [customize,update]
reward: true
---


# 自定义react更新机制

react以virtual dom和diff算法闻名。简单的说就是react会在内存中先根据组件先渲染出描述dom的虚拟dom。并且将虚拟dom与之前的虚拟dom对比差异，只渲染差异部分。这种做法能让开发者不关心dom操作，并且还有说得过去的性能。一般来说，我们是不需要去关注dom操作的。
<!-- more -->

但是情况都有例外的，思考一些特别的情况，比如计时器，而且是毫秒级别的那种，在页面上以毫秒级别更新的。又或者是自己实现滚动机制,要频繁的去更新Style的。比如这样:

## scrollContainer.js
```javascript
function ScrollDiv(props) {
    return (<div class="position_css"
        style={{
            top: props.top
        }}
    >
        {props.children}
    </div>)
}
//假设这样使用
<ScrollDiv top={this.state.top} onScroll={this.onScroll}>
    {
        map(list,item=><span>{item.name}</span>)
    }
</ScrollDiv>
```
那在top变化频繁的时候每次都会经过render,然后diff，再更新style里的top值。其实在我们知道该怎么更新的情况下，这套流程走下来多少有点浪费。可以改成：
```javascript
class ScrollDiv extends Components {
    shouldComponentUpdate(nextProps) {
        //自己去实现更新dom
        this.scroller.style.top = nextProps.top;

        //要阻止这个组件刷新
        //可以不早一直返回false但是不要让top的改变引起整个组件刷新
        return false
    }
    render() {
        return (<div class="position_css" ref={ref=>this.scroller = ref}>
            {props.children}
        </div>)
    }
}
```
不保证这个代码的质量，只是提供一种思路。以后能不借助react-iscroll这个不是很优秀的库也能做到自己的scoll。