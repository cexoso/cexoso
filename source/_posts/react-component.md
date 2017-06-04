---
title: react_component
date: 2017-05-20 19:30:41
tags:
---

# 重构示例 使用HOC编写可维护的React Component 

先上一个例子，是我公司历史上真正写出的代码。

# main.js
```javascript
import React from "react";
import { connect } from 'react-redux';
import * as mainActions from '../../reduces/manager/main';
import * as visitBarTypes from '../../actions/visitBar';
import * as orgActTypes from '../../../../public/container/orgTree';
import VisitBarChart, { getVisitParams, getAppendVisitData } from "../../../../public/bussiness/visitBarChart";
import getServerTime from "../../../../public/js/serverTime";
import FirstMenu from "../../../../public/bussiness/firstMenu";
import SecondMenu from "../../../../public/bussiness/secondMenu";
import TopMenuManager from "../../../../public/components/topMenuManager";
import BottomMenuManager from "../../../../public/components/bottomMenuManager";
import FollowStatistics from '../followStatistics';
import LfdjFollow from '../lfdjFollow';
import DealStatistics from '../dealStatistics';
import OverdueStatistics from '../overdueStatistics';
import Moment from "easy-datetime";
import {isSales} from '../../../../public/js/roleMode';
import classnames from 'classnames';

const timeDimensions = ['day', 'week', 'month', 'season', 'year'];
const now = getServerTime();
const today = new Date(new Moment(now).format('yyyy/MM/dd'));
class ManagerApp extends React.Component {
    constructor(props) {
        super(props);
    }
    componentDidMount() {
        const {main} = this.props;
        main.hasLoadData || this.loadStore();
    }
    loadStore() {
        const {userInfo, dispatch} = this.props;
        const apis = ['visitBarChart', 'followStatistics', 'dealStatistics', 'overdueStatistics', 'isErp'];
        if (userInfo.isManager) {
            apis.push('teamMenu');
        } else if (userInfo.isTeamer) {
            apis.push('groupMenu');
        }
        dispatch(mainActions.fetchMergeData(apis));
    }
    onSelectChange({date}) {
        const {dispatch} = this.props;
        dispatch(visitBarTypes.changeDate(date.getTime()));
        dispatch(mainActions.fetchMergeData(['followStatistics', 'dealStatistics']))
    }
    onDimensionChange({date_type, date}) {
        const {dispatch} = this.props;
        dispatch(visitBarTypes.changeDimension(date_type, date.getTime()))
        dispatch(mainActions.fetchMergeData(['visitBarChart', 'followStatistics', 'dealStatistics']));
    }
    onEdgeChange({dege}) {
        const {dispatch, userInfo, roleInfo, main} = this.props;
        const {date_type, date, store} = main;
        const list = store.visitBarChart.list;
        const params = getVisitParams({ userInfo, roleInfo, date_type, date, dege, list, needRefresh: false })
        if (new Moment(params.start_date) > today) {
            return
        }
        getAppendVisitData('gjjl-statistics/get-list-by-date', {
            params: getVisitParams({ userInfo, roleInfo, date_type, date, dege, list, needRefresh: false }),
            date_type
        }).then((data) => {
            data.list = dege ? list.concat(data.list) : data.list.concat(list);
            data.loading = false;
            dispatch(visitBarTypes.changeDege(data))
        })
    }
    changeTeam(team) {
        const {dispatch} = this.props;
        dispatch(orgActTypes.changeOrg(team));
        dispatch(mainActions.fetchMergeData(['visitBarChart', 'followStatistics', 'dealStatistics', 'overdueStatistics']))
    }
    checkFirst(team) {
        if (!(team.subs && team.subs.length > 0)) {
            this.changeTeam(team);
        }
    }
    checkSecond(team) {
        this.changeTeam(team);
    }
    toggleTeamModal() {
        const {dispatch} = this.props;
        dispatch(orgActTypes.toggleOrgModal());
    }
    onFilterTap(e) {
        e.preventDefault();
        const {dispatch} = this.props;
        dispatch(mainActions.saveStateToLocal());
        this.context.router.push({
            pathname: 'filter'
        });
    }
    renderMenu() {
        const {userInfo} = this.props;
        const {store, showOrgMenu} = this.props.main;
        let teamMenu = <div />;
        if (userInfo.isManager) {
            teamMenu = (<SecondMenu
                data={store.teamMenu}
                showModal={showOrgMenu}
                checkFirst={(team) => { this.checkFirst(team) }}
                checkSecond={(team) => { this.checkSecond(team) }}
                toggleModal={() => { this.toggleTeamModal() }}
            />)
        }

        if (userInfo.isTeamer) {
            teamMenu = (<FirstMenu
                data={store.groupMenu}
                showModal={showOrgMenu}
                checkFirst={(team) => { this.checkFirst(team) }}
                toggleModal={() => { this.toggleTeamModal() }}
            />)
        }
        if (userInfo.isGrouper) {
            teamMenu = (<FirstMenu
                data={[{ name: userInfo.team_name + '-' + userInfo.group_name, checked: true }]}
            />)
        }
        return (<div className="drop_selected bg_white">
            {teamMenu}
            <a className="search_select" onClick={this.onFilterTap.bind(this)}>
                <i className="iconfont icon-selectoff f20" />
            </a>
        </div>)

    }
    renderFollow() {
        const {dispatch, showModal, main} = this.props;
        const {store, bindLfdj} = main;
        return bindLfdj ? <LfdjFollow
            statistics={store.followStatistics}
            hasErp={store.isErp}
            showModal={(comp) => { showModal(comp) }}
            goDetail={(e, gjfs, cst_type, cstNum) => dispatch(mainActions.goFollowDetail(e, gjfs, cst_type, '', '', '', '', cstNum))} 
        />
            : <FollowStatistics
                statistics={store.followStatistics}
                hasErp={store.isErp}
                showModal={(comp) => { showModal(comp) }}
                goDetail={(e, gjfs, cst_type, cstNum) => dispatch(mainActions.goFollowDetail(e, gjfs, cst_type, '', '', '', '', cstNum))} 
            />
    }
    render() {
        const {dispatch, isReferer, showModal, main} = this.props;
        const {store, date_type, date} = main;
        return (
            <div>
                <div className={classnames("wrap bg_gray4", isSales() ? "" : "bottom_footmenu")} ref="wrap">
                    {!isReferer && <TopMenuManager current="statistics" />}
                    {!isReferer && this.renderMenu()}
                    <VisitBarChart
                        timeDimension={timeDimensions[date_type - 1]}
                        timeDimensions={timeDimensions}
                        date={new Date(date)}
                        date_type={date_type}
                        visitData={store.visitBarChart}
                        onDimensionChange={(data) => { this.onDimensionChange(data) }}
                        onEdgeChange={(data) => { this.onEdgeChange(data) }}
                        onSelectChange={(data) => { this.onSelectChange(data) }}
                    />
                    <div className="case_staticstics">
                        {this.renderFollow()}
                        <DealStatistics
                            deal={store.dealStatistics}
                            hasErp={store.isErp}
                            showModal={(comp) => { showModal(comp) }}
                            goDetail={(e, bizType) => dispatch(mainActions.goDealDetail(e, bizType))}
                        />
                        <OverdueStatistics
                            counts={store.overdueStatistics}
                            goDetail={(e, overdueType) => dispatch(mainActions.goOverdueDetail(e, overdueType))}
                        />
                    </div>
                </div>
                {!isSales() && <BottomMenuManager current="statistics" />}
            </div>
        )
    }
}
ManagerApp.contextTypes = {
    router: React.PropTypes.object.isRequired
}
function mapStateToProps(state) {
    return {
        userInfo: state.userRole.userInfo,
        roleInfo: state.userRole.roleInfo,
        main: state.main
    }
}
function mapDispatchToProps(dispatch) {
    return {
        dispatch
    }
}
export default connect(mapStateToProps, mapDispatchToProps)(ManagerApp)

```
如果你匆匆的瞄了一眼,应该会感觉上面的代码很乱是不是？

上面的代码有几个特点。


1.视图层和逻辑层混合在一起

>由于react提供class的写法。所有视图层的代码很理所当然的容易跟逻辑层的代码混写在一起。
这一点的好处是如果代码写得好紧凑的话，并不会显得难看。

2.多组件逻辑代码穿插

>造成无关逻辑代码穿插在一个文件中的原因是。这是容器组件，承载着这个页面逻辑。这也是为什么我选这个例子的原因。

3.代码细节，渲染细节暴露所以层次不分明

其它缺点不属于本次文章所谈范围。

## 再让我们看看怎么解决

你应该知道stateless组件，其实就是一个纯函数，输入什么就输出什么。输出的东西没有状态，所有的依赖都从props上注入。由于纯函数把逻辑都丢给上层去实现，所以写起来特别的简单。让我们试试怎么实现一个纯函数组件，然后慢慢的把纯函数变成能承载我们业务的组件。

假设我们
```javascripe
import React from "react";
function FollowStatistics({hasErp,showModal,goDetail,goDetail}) {
    return (<div>
        此处隐藏具体逻辑反正你们也不关心
    </div>)
}
function LfdjFollow({hasErp,showModal,goDetail,goDetail}) {
    return (<div>
        此处隐藏具体逻辑反正你们也不关心
    </div>)
}
```
接下来让我们实现这个逻辑
```javascripe
//已隐藏无关代码 
class main｛
    renderFollow() {
        return bindLfdj ? <LfdjFollow /> : <FollowStatistics/>
    }
}
```
我们可以这样做
```javascripe
//再写一个纯组件 而不是用类函数来实现
//这方便于后续的扩展 虽然现在感觉效果不大
//但这是方向 因为大类是没法去优化的
function CombineFollowStatistics() {
    return bindLfdj ? <LfdjFollow /> : <FollowStatistics/>
}
```
我们还可以这样做
使用[recompose](https://github.com/acdlite/recompose)
```javascripe
//这段代码应该配合上下文去理解的。这里的意思是
//props上的bindLfdj属性如果为true的话就渲染lfdjFollow否则渲染FollowStatistics
//这是一个HOC,属于recompose库的方法
branch(property("bindLfdj"), renderComponent(lfdjFollow))(FollowStatistics)
```

接下来我们给这个组件注入数据，最后的代码如下
```javascript
import FollowStatistics from "../components/followStatistics"
import lfdjFollow from "../components/lfdjFollow"
import {
    setDisplayName,
    compose,
    mapProps,
    branch,
    renderComponent
} from "recompose"
import { connect } from "react-redux";
import { bindActionCreators } from "redux";
import { partial, property, get } from "lodash"
import { method } from "../action"
function mapStateToProps(state) {
    const followStatistics = get(state, "case_statistics.main.followStatistics", {});
    const bindLfdj = get(state, "case_statistics.main.bindLfdj");
    return {
        statistics: followStatistics,
        bindLfdj
    }
}
const mapActionToProps = partial(bindActionCreators, {
    goDetail: method.goFollowDetailForMain
})
const emhance = compose(
    setDisplayName("combineFollowAndLfdj"),
    connect(mapStateToProps, mapActionToProps),
    branch(property("bindLfdj"), renderComponent(lfdjFollow))
)
export default emhance(FollowStatistics)
```
我们使用connect注入了数据
>{ followStatistics,bindLfdj }

和方法
>{ goDetail1 }

为了调试我们还会为组件设置一个名字
>setDisplayName("combineFollowAndLfdj")

我们就可以在主页面中直接了。最后重构的代码如下：
# main.js
```javascript
import React from "react";
import { ManageButtomMenu } from "../../../containers/bottomMenu"
import { ManageTopMenu } from "../../../containers/topMenu"
import { OuterWrap, Body, StaticsticsContainer } from "./wrap"
import Menu from "./menu"
import VisitBarChart from "./visitBarChart"
import CombineFollowStatistics from "./combineFollowStatistics"
import DealStatistics from "./dealStatistics"
import OverdueStatistics from "./overdueStatistics"
export default function ManagerApp() {
    return (
        <OuterWrap>
            <Body>
                <ManageTopMenu />
                <Menu />
                <VisitBarChart />
                <StaticsticsContainer>
                    <CombineFollowStatistics />
                    <DealStatistics />
                    <OverdueStatistics />
                </StaticsticsContainer>
            </Body>
            <ManageButtomMenu />
        </OuterWrap>
    )
}
```
逻辑不见了，页面结构清晰了。那些逻辑都各回各家各找各妈去了。这也符合编程里只做一件事的说法（虽然一件事不好定义是什么但是这件事尽量不要定义得很大，在这里这件事就是构建页面结构）

再看看main.js自己的逻辑代码到哪里去了
```javascript
import { connect } from 'react-redux';
import { withRouter } from "react-router"
import { withProps, compose, lifecycle } from "recompose"
import main from "./containers/main"
import { actionsCreator } from "./../../../../base/config/action"
import { method } from "./action"
import { bindActionCreators } from "redux"
import { createSelector } from "reselect"
import { property, flowRight } from "lodash"
import { withLocalStorageWhenUnload } from "./../../../../base/systen/withSystem"
const mapStateToProps = createSelector(
    [
        property("case_statistics.main.date"),
        property("case_statistics.main.date_type")
    ],
    (date, date_type) => ({ date, date_type })
)
function mapDispatchToProps(dispatch) {
    return bindActionCreators({
        ...method,
        ...actionsCreator
    }, dispatch)
}
const enhance = compose(
    withLocalStorageWhenUnload("manager/case_statistice", property("case_statistics")),
    connect(mapStateToProps, mapDispatchToProps),
    withRouter,
    withProps(({ location: { query: { user_type, date_type } } }) => {
        const isReferer = !!user_type && !!date_type
        return {
            isReferer
        }
    }),
    lifecycle({
        componentDidMount() {
            const { props: {
                getCurrentProjIsErp,
                getTeamMenu,
                getCurrentUserRole,
                getVisitBarChart,
                getFollowStatistics,
                getDealStatistics,
                getGroupMenu,
                getOverdueStatistics,
                date_type,
                date
        } } = this
            getCurrentProjIsErp()
            const UserRolePromise = getCurrentUserRole()
            UserRolePromise.then(flowRight(getGroupMenu, property("roleInfo.value")))
            UserRolePromise.then(() => {
                getFollowStatistics({ date_type, date })
                getVisitBarChart({ date_type, date })
                getDealStatistics({ date_type, date })
                getOverdueStatistics({ date_type, date })
            })
            getTeamMenu()
        }
    })
)
export default enhance(main)
```
再另一个文件中通过
withLocalStorageWhenUnload("manager/case_statistice", property("case_statistics"))
connect(mapStateToProps, mapDispatchToProps)
withRouter
withProps
lifecycle
多个HOC实现了数据的注入逻辑的实现，其中withLocalStorageWhenUnload是自己实现的HOC，作用在于页面卸载后将某些数据存到localstorage中。这种装饰形的写法可以很方便逻辑的拆卸。

最后我感觉本文写得有点烂。很多想表达的东西都没有表达出来。代码也很冗长。毕竟这是我的第三遍文章吧。