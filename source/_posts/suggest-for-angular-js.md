---
title: 一次重构angular1.x代码建议收集
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

以下是纯js建议跟 angular关系不有
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
