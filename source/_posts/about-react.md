---
title: react复杂组件
date: 2017-06-17 18:18:23
tags: [react,复杂组件,components]
reward: true
---

# react复杂组件

开门见山的说。我刚开始认为react写一个组件是非常简单的事。因为react好像就是为了组件设计而生的，React的应用就是由组件组合而成的。

<!-- more -->
然作为view层的react，在缺少状态管理工具如redux时,会在构建应用的时候非常无力，也会在做复杂组件的时候乏力。把复杂组件比喻成一个简单的应用就显而易见了。

首先来看看一个，一个好的React组件应该具备的点:

一. 状态映射形态

组件有什么表现方式？长什么样？都应该有可能转换成能够通过props来控制的情况。举例说明:
```javascript
// component
function Input(props) {
    return <input {...props} type="text"/>
}

<Input value="the value i want"/>
```
Input有能力通过给定value值来让input显示给定值。组件不一定一开始就做到有非常完备的属性能够让使用者控制表现形式，但是一定要有能力在将来添加上这种能力。一个反例就是：react-iscroll只是简单的包裹了iscoll，它是没有办法让你传入一个值来控制滚动的位置的(能够给一个初始化的值，但之后不能够控制了)。对于这种不能自己掌控的组件有时候我们会采取代替的方法（并不是很好，一旦违背了规则就有可能在将来为捷径埋单）。

二. 外界注入逻辑

组件能够有自身的逻辑，一旦外界决定要插手逻辑，要让他们能够找到能插手的余地。例如:
```javascript
class Input extends Components {
    constructor(props) {
        super(props);
        this.state = {
            value: ""
        }
    }
    onChange(e) {
        this.setState({value: e.target.value})
    }
    render() {
        return <input {...this.props} onChange={this.onChange.bind(this)} value={this.state.value}>
    }
}
```
这是一个不好的例子。Input组件自己完成了能输入能显示的逻辑。但是外界插手不了这个逻辑。稍微修改一下：
```javascript
class Input extends Components {
    constructor(props) {
        super(props);
        this.state = {
            value: ""
        }
    }
    onChange(e) {
        const change = this.props.onChange
        this.setState(change ? change(e) : {value: e.target.value})
    }
    render() {
        return <input {...this.props} onChange={this.onChange.bind(this)} value={this.state.value}>
    }
}
```
这种实现方式也不是很好，只是提供一下例子。

三. 外界能够掌控组件

外界能够掌控组件说的是外界能知道现在组件是什么状态，例如:
```javascript
class Logic extends Components {
    render() {
        return (<Form>
            <Select name="f1"/>
            <DropDown name="f2"/>
            <Tel name="f2"/>
        </Form>)
    } 
}
```
Logic类中使用了表单Form来管理Select,DropDown,Tel组件。Logic类平时并不管这些表单组件的逻辑。而是在需要的时候能够知道表单的状态如所有字段的值？是否校验通过？是否正在进行异步校验？

# 解决方法
暂时没有