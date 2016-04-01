---
layout: post
title: "Angular 筆記"
date: 2016-03-27 20:43:10 +0800
comments: true
categories: angular
---
最近有幸在專案上用到 angular，不過卡在非常尷尬的點...  
已經出 angular2 了，但專案是用 angular1..但還是來學習一下吧    

<!-- more -->

AngularJS 跟 Rails 一樣是 MVC 框架，  
可以透過 Controller 來處理 Model 和 View 的綁定。

Model 可以在 Controller 中進行處理  
並且透過 `雙向資料繫結 (Two Way Data-Binding)`，可以即時的互動!

# Angular Setting
```js
//app/asset/javascript/app.js

//angular.module
//Where we write pieces of our Angular application.//Makes our code more maintainable, testable, and readable. 
//Where we define dependencies for our app.

var app = angular.module('app', //設定 module 名稱，只能有一個
  [ 'ui.router',                //Dependency Injection 其他 module
    'angularFileUpload'
  ])
```

# ui-router config

可藉由 `ui-router` module 來自定前端的 router，要再另外 inject，因為不是內建的

```js
//#字號前面為原本的 router，後面則為 ui-router，也可以透過 HTML5mode 來把 # 字號去除掉，但就要小心不要跟原本的 router 衝到。

http://localhost:3000/#/posts/list
```

```js
app
.config(['$httpProvider', function($httpProvider){
	
	//防止 CSRF，rails 預設都必須帶 token，才可以發 post，防止外來的攻擊
  	var headers = $httpProvider.defaults.headers.common;
  	token_tag = document.querySelector("meta[name=csrf-token]");
  	if( token_tag ){ headers['X-CSRF-TOKEN'] = token_tag.content; };
  	headers['X-Requested-With'] = 'XMLHttpRequest';
  	
  	//另一種方式
  	//$httpProvider.defaults.headers.common['X-CSRF-TOKEN'] = $('meta[name=csrf-token]').attr('content');
}])
.config(
  [      '$stateProvider', '$urlRouterProvider',
  function($stateProvider,  $urlRouterProvider) {

  // For any unmatched url, redirect to /state1
  $urlRouterProvider.otherwise("/home");

  // Now set up the states
  $stateProvider
    .state('posts', {
      abstract: true,
      url:"/posts",   
      template: "<ui-view></ui-view>",
    })
    .state('posts.list', {
      url: "/list", //http://localhost:3000/#/posts/list
      templateUrl: "/posts/index.html",
      controller: 'PostIndexCrtl'
    })
    .state('posts.show', {
      url: "/show/:id", //http://localhost:3000/#/posts/show/1
      templateUrl: "/posts/show.html",
      controller: 'PostShowCrtl'
    })
}])
```

官方文件：  
[ui-router](https://github.com/angular-ui/ui-router)  
[ui-router github.io](https://angular-ui.github.io/ui-router/site/#/api/ui.router)

#Angular Controller

```js
//app/asset/javascript/Post/PostIndexCrtl.js
app
.controller('PostIndexCrtl', //設定 controller 名稱，只能有一個
  [       '$scope', '$http', //注入這個 controller 會用到的模組
  function($scope, $http){   //這裏在要注入一次到這個 function 裡面
  
  	//定義一個 function 可以透過 $http 發 get 拿回 json
    var getPost = function(){
        $http.get('/posts').success(function(data){
        		//定義變數 post = 傳回來的dat
                $scope.posts = data;
        });
    };
    
    //這裏馬上執行，因為一定要有 data 才能做其他事
    getPost()
   
   //排序
   $scope.sortKey = 'id';
   $scope.sort = function(keyname){
	    $scope.sortKey = keyname;
	    $scope.reverse = !$scope.reverse; //加驚嘆號可以將值變成 boolean
	}
	
	//checkboxs
	var selection = []
	var fruits = [
        '蘋果','西瓜','香蕉'
    ];
    
   //當點擊 checkbox 將值丟到 selection，取消則刪除
   var toggleSelection = function toggleSelection(fruit) {
   		//indexOf()找出目前點擊的位置，若沒有則為 -1
        var idx = $scope.selection.indexOf(fruit);         
        if (idx > -1) {
      	//splice(位置，刪除第幾個，新增資料)
          $scope.selection.splice(idx, 1);
        } else {
          $scope.selection.push(fruit);
        }
      };
      
   	//也可以將所有變數定義在最後面，方便管理
   $scope.selection = selection;
   $scope.fruits = fruits
   $scope.toggleSelection = toggleSelection
}])
```

#view
在 <html ng-app="app"> ， ng-app 用來宣告 angular 的範圍  
記得要載入 angular library才能使用  
惱人的 turbolink 記得拿掉 

`<ui-view></ui-view>`  
ui.router 的語法，用來顯示透過 angular 拉回來的 view ，有點像是上面的 yield 是拉 rails 裡的 view 一樣
範例就是會拉下面 index 的 view 進來

#Angluar View
```html
<!--public/posts/index.html-->
<div class="box">
  <div class="box-header">
    <div class='pull-right'>
    	<!-- ng-model 綁定 data 用來篩選-->
      <label>Any: <input ng-model="search.$"></label>
      <label>ID: <input ng-model="search.id"></label>
    </div>
  </div>
  <div class="box-body">
    <table class="table table-bordered table-hover">
      <thead>
        <tr>
        	<!-- ng-clik 觸發 controller 裡的 sort() function 並帶參數進去 -->
          <th ng-click="sort('id')" class='pointer'>ID
          <!-- ng-show true 顯示 flase 隱藏，相反為 ng-hide -->
          <span class="glyphicon" ng-show="sortKey=='id'" ng-class="{'glyphicon-chevron-up':reverse,'glyphicon-chevron-down':!reverse}"></span>
          </th>
          <th>名稱</th>
          <th>描述</th>
        </tr>
      </thead>
      <tbody>
      	<!-- ng-repeat 展開取回來的 array data -->
      	<!-- filter 篩選 -->
      	<!-- orderBy 排序 -->
        <tr ng-repeat="video in videos | filter:search:strict | orderBy:sortKey:reverse">
        	<td>{{video.id || 'hi'}}</td>
          <td>{{video.id}}</td>
          <td>{{video.title}}</td>
        </tr>
    </tbody>
    </table>
  </div>
</div>

<!--checkbox 多選項-->
<label ng-repeat="fruit in fruits">
  <input type="checkbox" name="selectedFruits[]" value="{{fruit}}" ng-checked="selection.indexOf(fruit) > -1"
    ng-click="toggleSelection(fruit)" >
    {{fruit}}
</label>

<pre>{{videos | json}}</pre>
```

#指令

* `ng-app` 用來宣告 angular 的範圍，也可放在 `body` 裡面。
* `ng-init` 可以直接在 view 中設定初始值，不過偏好管理還是直接設在 js 比較方便
* `ng-model` 設定綁定的變數
* `ng-click` 點下去觸發，設定好的function()  
* `ng-show ng-hide` 搭配 ng-model 綁定資料，就可以來根據各種情況來顯示會隱藏
* `ng-repeat`
 
```js
<p ng-repeat="post in posts">{{$index+1}}.{{name}}</p>

$index – 取出陣列中的index。(從0開始)
$first – 如果是第一個項目，就傳回true、否則為false。
$middle – 如果不是第一個和最後一個項目，就傳回true、否則為false。
$last – 如果是最後一個項目，就傳回true、否則為false。
```
* `ng-checkbox` 勾選框

```js
ng-true-value="YES" ng-false-value="NO"
可搭配這兩個設定 ture 和 false 時的值
```
* `ng-checked` 可以讓勾選框有 '選擇全部' 勾選或 '取消全部' 勾選的狀態
* `ng-options` 下拉式選單

```js
<select ng-model="Select1" ng-options="post.name for post in posts">
	<option value="">-- 請選擇 --</option>
</select>
  
```
* `ng-switch` 可以搭配 `ng-options` [ng-switch](http://ithelp.ithome.com.tw/question/10136011)
* `ng-change` 當資料改變會跟者執行
* `ng-include` 像是 partial 一樣可以 include 其他 view 進來

```js
<ng-include src="'/templates/search_bar.html'"></ng-include>
```
* `ng-src` 用來取代 html 中 img 標籤裡的 src 屬性 

由於 AngularJS 是整個網站載完後才開始編譯，使用 ng-src 可以解決還沒載入完 html 發生的錯誤。

* `ng-href`是用來取代html中 a 標籤裡的href屬性

如果Angular還沒載入或載入失敗，會直接讓href讀取到{{ }}的內容，
而a 標籤裡的href屬性無法辨識{{ }}，就會連結到404的錯誤頁面。

* `ng-bind` 綁定 model，避免 html 載入完畢，js 還沒載入導致畫面顯示 {{}} 的狀態
* `ng-cloak` 跟 ng-bind 類似，需要另外增加css，用來解決跨瀏覽器顯示的問題
* `ng-bind-template` 可以一次綁定多個 model
* `ng-class` 用 angular 設定 class
* `ng-form` ng-form允許有巢狀表單(nest forms)，就是在表單內再增加一個或多個form，而form則無法達到此需求。
* `$scope` 用來宣告變數，有點像是 js 的 `var` 或著 rails 的 `@` 
* `$watch ` [範例](http://jsbin.com/oHIXIqa/3/edit?html,js,output)

```js
$watch、ng-change、ng-click的比較

範例中$watch和ng-change的結果相同，
但使用$watch指令，watch的值還會傳回newValue,oldValue，
提供“改變的新數值”和“前一次改變的舊數值”。

$watch和ng-change是在改變內容值的時候，觸發事件，ng-click則只會在點擊輸入框時被執行。
```

`filter`  

```
{{ 'test' | json}}

currency  貨幣符號
date      將 Date 轉成指定格式的日期
filter    可以篩選陣列中的內容
json      json
limitTo   限制陣列或字串裡面的字元個數
number    可以為數值或指定小數以下位數
lowercase 將英文字串轉成英文小寫
uppercase 將英文字串轉成英文大寫
orderBy   可以將資料作排序
``` 

官方文件：  
[AngularJs](https://angularjs.org/)  
[angular-ui](https://angular-ui.github.io/bootstrap/)  
[ui-router](https://github.com/angular-ui/ui-router)    
[ui-router github.io](https://angular-ui.github.io/ui-router/site/#/api/ui.router)

參考資料：  
[入門AngularJS筆記](http://ithelp.ithome.com.tw/profile/share?id=20071512&page=4)  
[AngularJS 系列](http://programer-learn.blogspot.tw/p/angularjs.html)  
[AngularJS 學習筆記系列 ](http://sweeteason.pixnet.net/blog/post/40589830)  
[給 Rails Developer 看的 Angular + Rails 起步走](http://carolhsu.github.io/blog/2014/06/07/step-by-step-with-angularjs-and-rails/)  
[Search Sort and Pagination in ng-repeat – AngularJS](http://code.ciphertrick.com/2015/06/01/search-sort-and-pagination-ngrepeat-angularjs/)    
[Angular 中文社區](http://www.angularjs.cn/T001)  
[[AngularJs]淺談Angular.js的Provider機制](http://kirkchen.logdown.com/posts/245678-angularjs-talking-about-the-angularjs-provider-mechanisms)  
[ANGULARJS $Q DEFERRED 和 PROMISE](http://blog.begin.com.tw/?p=30)  
[流浪貓の窩 - angular學習筆記](http://www.cnblogs.com/liulangmao/tag/angular/default.html?page=4)  
[NG筆記1-AngularJS學習資源與術語](http://blog.darkthread.net/post-2014-06-11-angularjs-notes-1.aspx)  
[學習 AngularJS](https://github.com/jmcunningham/AngularJS-Learning/blob/master/ZH-TW.md)  
[初學 AngularJS 的路](http://ithelp.ithome.com.tw/profile/share?id=20091340&page=2)

