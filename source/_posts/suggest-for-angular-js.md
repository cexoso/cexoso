---
title: 一次重构angular代码建议收集
date: 2017-07-17 16:24:31
tags:
---

# 一次重构angular1.x代码建议收集（这可能是一篇过时的文章了）

### html中没用到的变量不要挂载在$scope上
```javascript
$scope.getQueryParams = function () {};//错误
function getQueryParams() {}//正确
```
`getQueryParams`只在js文件中被调用,就不应该挂在$scope上。
<!-- more -->
### 优化篇
```javascript
$scope.cancelVerify = function (contract) {
    util.confirm('确定要撤销复核吗?', {
        ok() {
            const post_data = { contractId: contract.contract_id };
            const url = api_prefix + "/contract/cancel-review";
            util.myAjax({
                url,
                type: "POST",
                context: this,
                data: post_data,
                success(json) {
                    if (json.retCode === "0") {
                        contract.contract_status = 0;
                        $scope.$applyAsync();

                    } else {
                        util.alert("撤销复核失败：" + json.errMsg);
                    }
                }
            });
        },
        cancel() {
        }
    });
};
//post_data和url的定义不是很有必要 cancel函数没必要存在,异步函数应该使用promise来处理而不是回调
//修改如下
$scope.cancelVerify = function (contract) {
    util.confirm('确定要撤销复核吗?')
        .then(function whenOK() {
            return util.myAjax({
                url: api_prefix + "/contract/cancel-review",
                type: "POST",
                context: this,
                data: { contractId: contract.contract_id },
            })
        })
        .then(function whenReceipt(json) {
            if (json.retCode === "0") {
                contract.contract_status = 0;
                $scope.$applyAsync();
            } else {
                util.alert("撤销复核失败：" + json.errMsg);
            }
        })
};
//前提是util的方法支持promise而不只只是回调
//更正确的做法是将请求移到服务中而不应该在控制器中.这样逻辑代码不会受到非逻辑代码的干扰
//而错误处理并没有特别业务项的需求应该在拦截器中统一处理，不应该出现在这里
//应该尽量使用$http作为低层服务这样可以去掉非业务向的代码$scope.$applyAsync();
$scope.cancelVerify = function (contract) {
    util.confirm('确定要撤销复核吗?')
        .then(function whenOK() {
            return cancelReview({ contractId: contract.contract_id })
        })
        .then(function whenReceipt(json) {
            contract.contract_status = 0;
        })
};
```

以下是纯js建议跟 angular关系不大
### 将代码写到你不能再优化为止
```javascript
$scope.changeTimeType = timeType => {
    if (timeType.value !== 0) {
        $scope.showTimeSelected = 2;
    } else {
        $scope.showTimeSelected = 0;
    }
};//啰嗦
$scope.changeTimeType = ({value}) => $scope.showTimeSelected = value !== 0 ? 2 : 0;
```
#### 以下代码一定是CV战士写的。居然还懂得用$.grep的方法
```javascript
//收集过滤条件
//合同状态
var filterStatus = $.grep($scope.filterDataStatus, function (o, n) {
    return o.selected == true;
});
if (filterStatus.length) {
    parms.status = filterStatus[0].value;
}
//合同性质
var filterNature = $.grep($scope.filterDataNature, function (o, n) {
    return o.selected == true;
});
if (filterNature.length) {
    parms.property = filterNature[0].property_id;
}
//城市区域
var filterCity = $.grep($scope.filterDataCities, function (o, n) {
    return o.selected == true;
});
if (filterCity.length) {
    parms.areaId = filterCity[0].areaId;
}
//报表类型
var filterStatement = $.grep($scope.filterStatementType, function (o, n) {
    return o.selected == true;
});
if (filterStatement.length) {
    parms.reportType = filterStatement[0].value;
}
//时间类型
var filterTimeType = $.grep($scope.filterTimeType, function (o, n) {
    return o.selected == true;
});
if (filterTimeType.length) {
    parms.timeType = filterTimeType[0].value;
}
```
#### 重构后还是`don't repeat yourself`
```javascript
parms.status = chain($scope.filterDataStatus).find("selected").get("value").value()
parms.property = chain($scope.filterDataNature).find("selected").get("property_id").value()
parms.areaId = chain($scope.filterDataCities).find("selected").get("areaId").value()
parms.reportType = chain($scope.filterStatementType).find("selected").get("value").value()
parms.timeType = chain($scope.filterTimeType).find("selected").get("value").value()
```
图中的`chain`是`lodash`中的方法，不吹不黑，`lodash`在数据处理方面还是比`jquery`高不是一个档次的。如果不是名字瞎命名的话，一个琏式表达式就能解决了。
### 去掉无用的注释代码 
```javascript
//下拉框处理
//$scope.ddlConfigStrikeType = {
//    allowClear: false,
//    data: [],
//    placeholder: '时间类型'
//};
////时间类型
//$scope.ddlConfigStrikeType.data = [
//    {id: 1, text: '签约日期'},
//    {id: 2, text: '录入日期'},
//    {id: 3, text: '签约业绩所属日期'},
//    {id: 4, text: '汇款业绩所属日期'}]
```
以上注释只是被当作了历史的记录工具存在。而更好的方法去代替这段代码的的是去学习使用git。注释应该去掉。
```javascript
//操作
//提交复核
$scope.submitVerify = function (contract) {};
//取消复核
$scope.cancelVerify = function (contract) {};
//确认复核
$scope.confirmVerify = function (contract) {};
//作废
$scope.cancelContract = function (contract) {};
//删除
$scope.deleteContract = function (contract) {};
```
以上注释并不能提供更多理解代码的信息。一个好的函数签名就可以表明意图。注释应该去掉。

```javascript
/**
* 判断查询时间
* @param a
* @param b
* @returns {boolean}
*/
const compareDate = (a, b) => {
    const arrA = a.split("-");
    const start = new Date(arrA[0], arrA[1], arrA[2]);
    const startTime = start.getTime();

    const arrB = b.split("-");
    const end = new Date(arrB[0], arrB[1], arrB[2]);
    const endTime = end.getTime();

    if (startTime > endTime) {
        return true
    } else {
        return false
    }
};
```
以上代码 注释部分是完全多余的,而该注释的地方却没有注释出来。
参数的名称也非常的晦涩。
日期本身就可以比较。
存在部分重复的代码。
关于最后的`if else`只能说，精益求精。

重构后
```javascript
const toDate = str => {
    const arr = str.split("-");
    return new Date(arr[0], arr[1], arr[2]);
}
//dateStr1 and dateStr2 is a formatted string like yyyy-MM-dd
const compareDate = (startDate, endDate) => toDate(startDate) > toDate(endDate)
```
缺点是多暴露了一个`toDate`函数，这一点可以用模块化或者闭包的方法去解决。

### 吐槽
看到有`return o.selected === true;`的代码。js虽然是弱类型语法，但是如果严格一点，能确保`selected`是`bool值`的话 完全可以写成`return o.selected`。

## 次日新增如下内容

```javascript
const appPrices = [];
$.each(item.applicationPrices || [], function (i, d) {
    appPrices.push({
        id: d.price,
        text: d.unit
    });
});

$dialogScope.ddlConfigSMSPrice = {
    allowClear: false,
    data: appPrices,
    placeholder: '标准单价',
    onchange2(price) {
        $dialogScope.data.unit = price.text;
    }
};
```
使用`lodash`重构如下
```javascript
import { map } from 'lodash'
$dialogScope.ddlConfigSMSPrice = {
    allowClear: false,
    data: map(item.applicationPrices , ({ price,unit })=>({
        id: price,
        text: unit
    })),
    placeholder: '标准单价',
    onchange2(price) {
        $dialogScope.data.unit = price.text;
    }
};
```
`map`方式总会返回正确的类型。不需要花哨的代码去判断`undefined`。

类似代码还有如下
```javascript
const ddlData = [];
let isCompanyValid = false;
$.each(data, function (i, d) {
    ddlData.push({
        id: d.company_id,
        text: d.company_name
    });
    if (d.company_id === $scope.contractBaseInfo.company_id) {
        isCompanyValid = true;
    }
});
$scope.ddlConfigCompany.data = ddlData;
```
理论上我觉得`isCompanyValid`与`ddlData`应该分别处理而不应该合在一起处理的
```javascript
const isCompanyValid = findIndex(data,['company_id',$scope.contractBaseInfo.company_id]) !== -1
$scope.ddlConfigCompany.data = map(data, ({ company_id, company_name }) => ({
    id: company_id,
    text: company_name
}));
```
`findIndex(data,['company_id',$scope.contractBaseInfo.company_id])`这个函数会处理data里面的每一个元素，找到属性`company_id`为`$scope.contractBaseInfo.company_id`的元素下标并返回，如果找不到就返回`-1`。虽然的函数的签名很奇怪，但是读懂后使用会很方便。

部分人可能觉得性能会有问题。个人觉得这点小性能实在不应该成为代码规范的拦路石。如果实在有强迫症的人可以采取折中的方法。
```javascript
let isCompanyValid = false
$scope.ddlConfigCompany.data = map(data, ({ company_id, company_name }) => {
    isCompanyValid = company_id === $scope.contractBaseInfo.company_id
    return {
        id: company_id,
        text: company_name
    }
});
```
这可能违反了`只做一件事的原则`，当一个函数做了两件事三件事更多件事的时候。我们在处理一件事的时候就会被其它的事干扰到。而人的精力是有限的。被干扰的后果是不知不觉的引进了`bug`所以保证你的函数简单是最好的方法。

有的人可能觉得函数编程比较难看懂，比如`const isCompanyValid = findIndex(data,['company_id',$scope.contractBaseInfo.company_id]) !== -1`。那是因为一句话做了太多的事情了。
```javascript
const includesWithPropertyEq = compose(eq(-1),findIndex);
const isCompanyValid = includesWithPropertyEq(data,['company_id',$scope.contractBaseInfo.company_id]);
```
利用函数的组合可以把多件事情组合成一个函数。然后通过命名让别人知道你的函数是作用。（以上代码假装`eq`是`柯里化`的

```javascript
const ddlData = [];
// 隐藏
// xx签约-联营区域:7f828b1b-fce8-11e4-bed8-00155d02c832,
// 联营签约-xx结算:7f84cfad-fce8-11e4-bed8-00155d02c832
$.each(data, function (i, d) {
    if (d.property_id != '7f84cfad-fce8-11e4-bed8-00155d02c832') {
        ddlData.push({
            id: d.property_id,
            text: d.property_name
        });
    }

});
$scope.ddlConfigContractNature.data = ddlData;
```
代码本身有注释非常好，让人知道那一串`7f84cfad-fce8-11e4-bed8-00155d02c832`是个什么玩意。如果没有注释的话一定要记得取个好名字比如。
`const aGoodName = '7f84cfad-fce8-11e4-bed8-00155d02c832'`（示例而已，你自己就不要起`aGoodName`了）

但是代码本身做的事其实就是`filter`,或者`reject`（`reject`与`filter`是反逻辑的）,所以：

```javascript
// 隐藏
// xx签约-联营区域:7f828b1b-fce8-11e4-bed8-00155d02c832,
// 联营签约-xx结算:7f84cfad-fce8-11e4-bed8-00155d02c832
$scope.ddlConfigContractNature.data = reject(data,['property_id','7f84cfad-fce8-11e4-bed8-00155d02c832'])
```
如果你还记得上面那个奇怪的签名。那也能理解这个签名。
> `reject(data,['property_id','7f84cfad-fce8-11e4-bed8-00155d02c832']`的意思是把`data`里面，属性`property_id`为`7f84cfad-fce8-11e4-bed8-00155d02c832`的元素去掉。与`reject(data,({property_id})=>property_id === '7f84cfad-fce8-11e4-bed8-00155d02c832')`是等价的，`filter(data,({property_id})=>property_id !== '7f84cfad-fce8-11e4-bed8-00155d02c832')`实现了相同的功能。（因为`reject`与`filter`是反逻辑的）

### 上面的代码还是有问题的，因为原代码做了一个命名的转换，但是我们没有做。
修改后如下
```javascript
$scope.ddlConfigContractNature.data = chain(data)
                .reject(['property_id', '7f84cfad-fce8-11e4-bed8-00155d02c832'])
                .map(({ property_id, property_name }) => ({ id: property_id, text: property_name }))
                .value();
```
`chain`做了打包，是一个很好的琏式调用，记录最后要用value来解包

**不管怎么说，能少写一行代码 就少写一行代码**

虽然原代码，多看一下也能理解，但是大量的更容易理解的代码与多看一眼能理解的代码，对人的影响就绝不是多看一眼的影响了。