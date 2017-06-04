---
title: multi-level-component
date: 2017-06-04 08:05:20
tags:
---
# 多层次的组件
让我们从一个示例开始
## 有这样一个需求：我们需要一个列表展示页。
```javascript
import React,{ Component } from "react"
import { map } from "lodash"
export default class List extends Component {
    render() {
        const { list } = this.props
        return (<ul>
            {
                map(list,item => (<li>
                    {item.name}
                </li>))
            }
        </ul>)
    }
}
```
尽乎完美，因为逻辑简单，需求明确。以至于不需要任何注释。
## 接下来我们需要隐藏一些数据，list里面的数据带有disable的就不要展示了
```javascript
import React,{ Component } from "react"
import { chain } from "lodash"
export default class List extends Component {
    render() {
        const { list } = this.props
        return (<ul>
            {
                chain(list)
                    .reject(item=>item.disable)
                    .map(item=>(<li>
                        {item.name}
                    </li>))
                    .value()
            }
        </ul>)
    }
}
```

利用lodash的chain重构一下整个逻辑还不算太坏。而且列表拥有是否显示数据的功能感觉也是情理之中的要求。

> 小提示： reject函数与filter函数逻辑相反,filter函数会根据返回值是否为真而保留数据，而reject函数会根据返回值是否为真而丢弃数据。而**item=>item.disable** 的形式可以用 **property("disabled")** 来代替。而reject的模式匹配中本身就支持property的调用所以最终代码可以写成 **reject("disable")**

其实代码中已经有坏味道了。但是本着写"hello world"不需要设计模式的思想。我们还hold得住就不去重构它。

## 接着来，我们需要在没有数据的时候使用"has not data"样式展示
直接动手
```javascript
import React,{ Component } from "react"
import { chain } from "lodash"
export default class List extends Component {
    renderNotData() {
        return <div>没有数据可供展示</div>
    }
    rendderList() {
        const { list } = this.props
        return (<ul>
            {
                chain(list)
                    .reject("disable")
                    .map(item=>(<li>
                        {item.name}
                    </li>))
                    .value()
            }
        </ul>)
    }
    render() {
        const { list } = this.props
        return chain(list).reject("disable").size().eq(0).value() ?
                    renderNotData() : rendderList()
    }
}
```
或者
```javascript
import React,{ Component } from "react"
import { chain } from "lodash"
import NoData from "./noData"
export default class List extends Component {
    hasData() {
        // TODO
    }
    render() {
        const { list } = this.props
        return this.hasData() ? <ul>
            {
                chain(list)
                    .reject(item=>item.disable)
                    .map(item=>(<li>
                        {item.name}
                    </li>))
                    .value()
            }
        </ul> : 
        <NoData />
    }
}
```
现在代码逻辑已经有一点复杂了。无论如何这段代码应该都不是一眼看穿的了。
而这不是最后的形态，我们还可能继续被提需求。例如：
* 增加一个hasFetch属性，表示是否进行了数据请求，如果没有进行请求时展示引导界面。
* 在列表善增加一个tips来展示某些信息

等等

-----

我觉得此时造成问题的原因已经很暴露了。答案就是类的责任范围被放大了。做了自己不应该做的事。回想当初，我们只是想写一个简单的，纯粹的List组件。而解决这些问题的方法就是 **只做一件事**

比如展示空数据的需求只需要再写一个类包裹住List组件就好了。
#### maybeList.js
```javascript
import React,{ Component } from "react"
import { chain } from "lodash"
import List from "./list"
export default class MaybeList extends Component {
    hasData() {
        // TODO
    }
    render() {
        const { list } = this.props
        return this.hasData() ? <List {...this.props}/> : <NoData />
    }
}
```
这样这个类只控制是否展示无数据界面。而怎么展示列表界面的职责还是委托给List去做了。但是这个类还是有问题。因为他依赖了List。我只不过想要做一个是否展示无数据的逻辑。可是我莫名其妙的得到了一个List，就好想我想做一个容器却给一块蛋糕一样。可以用HOC(高阶组件)来解决。

来一个HOC来阐述一下原理。
#### withNotData.js
```javascript
import React,{ Component } from "react"
function withNotData(hasDataFn,NoData) {
    return function (Bewrap) {
        return class _wrap extends Component {
            render() {
                return hasDataFn(this.props) ? 
                        <Bewrap {...this.props}/> : 
                        <NoData {...this.props}/>
            }
        }
    }
}
```

再写简单点
```javascript
import React,{ Component } from "react"
function withNotData(hasDataFn,NoData) {
    return function (BeWrap) {
        return function _wrap(props) {
            return hasDataFn(props) ? <BeWrap {...props}/> : <NoData {...props}/>
        }
    }
}
```
或者
```javascript
const withNotData = (hasDataFn,NoData) => 
                    BeWrap => props => 
                    hasDataFn(props) ? <BeWrap {...props}/> : <NoData {...props}/>
```

使用说明如下
```javascript
import { isEmpty } from "lodash"
import ANotDataComponent from "./aNotDataComponent"
export cosnt maybeList = withNotData(props=>isEmpty(props.list),ANotDataComponent)(List);
```

### **recompose**
[acdlite/recompose](https://github.com/acdlite/recompose) 是一个HOC库。之前的代码可以使用recompose来完成：

```javascript
import { isEmpty } from "lodash"
import { branch,renderComponent } from "recompose"
import ANotDataComponent from "./aNotDataComponent"
const enhance = branch(props=>isEmpty(props.list),renderComponent(ANotDataComponent))
export cosnt maybeList = enhance(list);
```

> 小提示： props=>isEmpty(props.list) 本意是取list属性再判断list属于是不是为空，可以用函数组合的方式完成**isEmpty·property("list")** ,javascript中可以借助lodash完成: **flowRight(isEmpty,property("list"))**

例子讲完了，我们来讲道理。

# 为什么要用HOC
首先一个React应用应该是由绝大部分 function components 构成的。
```javascript
const Greeting = props =>
  <p>
    Hello, {props.name}!
  </p>
```
function components 有几点好处：
* 杜绝了setState的滥用，更多是用propsw代替
* 代码的重用性更好,更模块化
* 鼓励使用组合的方式来完成一个大应用
* 为将来的性能优化留有余地

有一个矛盾是function components中看不到业务的代码，而上层组件又想尽量少的看见组件实现的细节。那这一部分逻辑就应该在组合链中做。举个例子：

#### 业务文件 app.js
```javascript
import React from "react"
import TextArea from "./textArea"
import Container from "./container"
export default function App() {
    return (<Container>
        <TextArea />
    </Container>)
}
```
代码中并没有为TextArea传递任何的props。这样减少细节而更专注功能。
#### function components　input.js
```javascript
import React from "react"
export default function Input(props) {
    return <input {...props}/>
}
```
一个几乎什么都没做的代码。但是Input是TextArea的原型。接下来:
#### textArea.js
```javascript
import Input from "./input";
import { compose } from "recompose";
const TextArea = compose(
    //逻辑代码放这里
)(Input);
export default TextArea;
```
首先我们要把TextArea与store连接
```javascript
const withConnect = connect(
    state=>({
        value: state.value
    }),
    dispatch=>bindActionCreators({
        changeValue
    },dispatch)
)
const TextArea = compose(
    withConnect
)(Input);
```
我们先为Input把默认的我们不再关心的props给传递一下
```javascript
const withDefaultProps = withProps({
    type: "text",
    placeholder: "请输入",
})
const TextArea = compose(
    withConnect，
    withDefaultProps
)(Input);
```
我们再为input加上onChange
```javascript
const withOnChangeHandle = withHandle({
    onChange:　props=>e=>props.changeValue(e.target.value)
})
const TextArea = compose(
    withConnect，
    withOnChangeHandle,
    withDefaultProps
)(Input);
```
其中props上的changeValue是由上层的withConnect传递下来的一个绑定了dispatch的方法,此时我们已经得到了一个把会dispatch的，绑定了store里某个值的TextArea。如果你觉得props不够清晰可以显示的mapProps一下
```javascript
const filterProps = mapProps(props=>({
    onChange:　props.onChange,
    value: props.value
}))
const TextArea = compose(
    withConnect，
    withOnChangeHandle,
    filterProps,
    withDefaultProps
)(Input);
```
这样input最终接收到的东西只有onChange value type placeholder。connect注入里面的dispatch方法在filterProps的时候被过滤掉了。最终我们即保证了上层业务代码的简单(使用TextArea不需要维护任何props)，又保证了低层Input组件的纯粹（一个几乎什么都没做的代码）。recompose构建了像一个从通用到专用之前的通道。承载着细节的业务逻辑。

假如我们以后需要修改业务代码，让TextArea只能输入数字，只需要修改对应的withOnChangeHandle就可以了，保证不会影响到其它使用了input的代码。