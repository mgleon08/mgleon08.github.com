---
layout: post
title: "Angular 筆記2"
date: 2016-04-19 22:12:20 +0800
comments: true
categories: javascript angular
---

一些 angular 的紀錄!

<!-- more -->

#vaildation

`novalidate` `required` Turn Off Default HTML Validation

```js
￼<form name="reviewForm" ng-controller="ReviewController as reviewCtrl"                        ng-submit="reviewForm.$valid && reviewCtrl.addReview(product)" novalidate><select ng-model="reviewCtrl.review.stars" required> <option value="1">1 star</option> ￼￼...  </select>Mark Required Fields￼￼<textarea name="body" ng-model="reviewCtrl.review.body" required></textarea><label>by:</label><input name="author" ng-model="reviewCtrl.review.author" type="email" required/>￼<div> reviewForm is {{reviewForm.$valid}} </div><input type="submit" value="Submit" />  </form>
```

---

#AngularJS q deferred 和 promise

[AngularJS q deferred 和 promise](http://roxannera.blogspot.tw/2014/03/angularjs-q-deferred-promise.html)    
[廣義回調管理](https://checkcheckzz.gitbooks.io/angularjs-learning-notes/content/chapter11/11-2.html)


---

#Factory Service Provider

`$factory` `$service` `$provider`
這三個都能夠做到同樣的事情，有點像是 rails 裡的 service object

差別在於

###factory 方法  
必須提供一個工廠方法，並自己建立一個 Service Object

```js
var app = angular.module("app", []);

app.service("helloService", function(){
     return {
        foo: function(name){
        return console.log('Hi' + name);
     }
});
```

###service 方法
必須提供一個建構子，由 AngularJS 會利用 new，建立 Service Object

```js
var app = angular.module("app", []);

app.service("helloService", function(){
    this.foo = function(name){
        return console.log('Hi' + name);
    };

});
```
###Provider 方法
與其他兩個最大的不同在於它能夠對 Service Object 做額外的設定(Configured)，且必須包含一個 `$get` 的函式

```js
var app = angular.module("app", []);

app.provider("helloService", function(){
return {  //It's Provider Object
    $get: function(){
            return {  //It's Service Object
               foo: function(name){
                   console.log('Hi' + name);
               }
            };
        }
    }
});
```

###Decorator
可以攔截並且去做客製化

```js
angular.module('app', []);

angular.module('app')
    .value('version', 1);

angular.module('app')
    .config(function ($provide) {
    $provide.decorator('version', function ($delegate) {
        return 'Version - ' + $delegate;
    });
});

angular.module('app')
    .controller('MainCtrl', function ($scope, version) {
    $scope.version = version;
});

//=> Version - 1
```

###Constant
常數，無法透過 decorator 去攔截  
無法修改  
可被注入到 Config

```js
angualr.module('app')
    .constant('version', 1);
```

###Value
可透過 decorator 去攔截  
可修改   
可被注入到 Config

```js
angular.module('app')
    .value('version', 1);
```

官方文件：  
[provide](https://docs.angularjs.org/api/auto/service/$provide)

參考文件：  
[AngularJS 系列](http://programer-learn.blogspot.tw/p/angularjs.html)  
[淺談Angular.js的Provider機制](http://kirkchen.logdown.com/posts/245678-angularjs-talking-about-the-angularjs-provider-mechanisms)  
[[AngularJS系列(4)] 那傷不起的provider們啊~ (Provider, Value, Constant, Service, Factory, Decorator)](http://hellobug.github.io/blog/angularjs-providers/)  
[Differences Between Providers In AngularJS](http://blog.xebia.com/differences-between-providers-in-angularjs/)  

---
#$rootScope $emit $broadcast $on

`$rootScope` 是 AngularJS app 中最上層的 scope，一個 app(ng-app) 只會有一個 `$rootScope`，也就是說同一個 app 可以共用同一個 `$rootScope` 的值

可以看作 

* `$scope` 區域變數。  
* `$rootScope` 全域變數。

可以搭配 `$emit`(向父級) , `$broadcast`(向子級) and `$on`(接收) ，跟別的 controller 溝通

參考文件：  
[angularjs的事件$broadcast and $emit and $on](http://www.angularjs.cn/A08c)  
[AngularJS的學習--on、emit和broadcast的使用](http://www.cnblogs.com/CraryPrimitiveMan/p/3679552.html)  
[$emit , $broadcast and $on的用途 -- AngularJS](http://hamisme.blogspot.tw/2013/07/emit-broadcast-and-on.html)

---

#ngResource

[AngularJS 取用外部伺服之：取用 RESTful APIs](http://jasonlamswatow.com/angularjs-restful/)  

---

#理解watch ，apply 和digest
[理解watch ，apply 和digest --- 理解數據綁定過程](http://www.angularjs.cn/A0a6)
