---
layout: post
title: "Angular Custom Directive"
date: 2016-03-27 21:07:16 +0800
comments: true
categories: angular
---

在 angular 當中可以自己客製 Directive，讓 code 變得更乾淨。

<!-- more -->

```js
app.directive('productTitle', function(){
	return{
		restrict: 'E'
		templateUrl: 'product-title.html'
	}
});

<product-title></product-title>

app.directive('productTitle', function(){
	return{
		restrict: 'A'
		templateUrl: 'product-title.html'
	}
});

//js 的 productTitle 名稱在 html 就會變成 product-title
<h3 product-title></h3>

‘A’ – Attribute (You want to use your directive as <div rating>)
‘E’ – Element (Use it as <rating>)
‘C’ – Class. (Use it like <div class=”rating”>)
‘M’ – Comment (Use it like <!– directive: rating –>
```

`controller` 一起包在 `app.directive` 裡面

```js
app.directive('productTitle', function(){
	return{
		restrict: 'A'
		templateUrl: 'product-title.html'
		controller: function() {
    		this.tab = 1;
    		},
    	controllerAs: "tab"
	}
});

//js 的 productTitle 名稱在 html 就會變成 product-title
<h3 product-title></h3>
```

參考文件：  
[入門AngularJS筆記-directive
](http://ithelp.ithome.com.tw/question/10140689)  
[自訂 AngularJs Directive (1) :開始](http://exile1030-blog.logdown.com/posts/167848-custom-angularjs-directive-1)  
[自訂 AngularJs Directive (2): Hello World](http://exile1030-blog.logdown.com/posts/168283-customizing-the-angularjs-directive-2)