---
layout: post
title: "Javascript 筆記"
date: 2016-04-19 22:12:34 +0800
comments: true
categories: javascript
---

javascript 的一些紀錄!

<!-- more -->

#Closure
Closure 閉包

一般來說每個變數都會生存在自己的 scope 裡面， function 裡面只能用到裡面定義好的 `Local Variable`，但在 javascript 卻能夠動用到外部的 variable

```js
var x = 8;
funtion closure1(){
	console.log(x)
}
closure1();
//=> 8
```
在 closure1 雖然 function 裡並沒有 var x 但卻輸出外面的 8  
主要是因為在 javascript 當中，在 local 中找不到 x 值的話，就會繼續往上一層去找  

```js
var x = 8;
funtion closure2(){
	var x = 5;
	console.log(x)
}
closure2();
//=> 5
```
closure2 則是在 local 就找到了 x 所以輸出的就是 5。

[用 Node.js 學 JavaScript 語言（3）函數、參數與閉包](http://www.codedata.com.tw/javascript/using-nodejs-to-learn-javascript-3-function-parameter-closure)  
[閉包（Closure）](http://openhome.cc/Gossip/JavaScript/Closure.html)

#Hoisting

```js
function test(x, y){
  if (x == y) {
    foo();
  }
  else{
    bar();
  }
  var function foo(){
    alert('FOO');
    return false;
  }
  var function bar(){
    alert('BAR');
    return true;
  }

}
```

```js
var x = 8;

function hoisting1(){
	console.log(x)
	var x = 5;
}

hoisting1()

//=> undefined
```
照上面的 `closure` 特性，應該輸出的要是 8 才對，因為裡面的宣告是放在下面  
可是卻輸出的是 `undefined`

這就是另一個 `hoisting` 的特性  
`hoisting` 會將宣告提升到 scope 的頂端，因此上面的程式應該是

```js
var x = 8;

function hoisting2(){
	var x;	
	console.log(x);
	x = 5;
}
hoisting2()
//=> undefined
```
看到上面多出一個 `var x;` 就是 `hoisting` 提升的，因此下面的 `console.log(x)` 才會出現 `undefined`  

要解決有兩種方式

```js
var x = 8;

function hoisting3(){
	var x = 5;
	console.log(x);
}
hoisting3()
//=> 5
```
```js
//var x = 8; 要拿掉這行否則會 call 到它 

function hoisting4(){
	console.log(x);
	x = 5;
}

hoisting4()
//=> 5
```

[Javascript Hoisting 與 Closure](http://www.puritys.me/docs-blog/article-242-Javascript-Hoisting-%E8%88%87-Closure.html)

#callback
[Javascript callbacks](http://dreamerslab.com/blog/tw/javascript-callbacks/)  
[【javascript】了解函式(function)很重要的筆記](http://fireqqtw.logdown.com/posts/258823-javascript-function-notes)  
[JavaScript callback function 理解](http://mao.li/javascript/javascript-callback-function/)  
[回呼函式(callback function)](http://www.victsao.com/blog/81-javascript/292-javascript-function-callback)

#JavaScript Memory_leaks
[Mozilla - Memory_leaks](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/%E9%87%8D%E6%96%B0%E4%BB%8B%E7%B4%B9_JavaScript#Memory_leaks)  
[JavaScript 記憶體洩漏（Memory Leak）問題](http://blogger.gtwang.org/2014/01/javascript-memory-leak-patterns.html)  
[javascript: Memory Leaks 的情況以及如何解決與偵測](http://blog.smlsun.com/2013/12/javascript-memory-leaks_3701.html)  
[Memory management in JavaScript](http://javascript.info/tutorial/memory-leaks#memory-management-in-javascript)

---

#PROTOTYPE
#￼OBJECT! PROTOTYPE

```js
constructor()
valueOf()
toLocaleString()
isPrototypeOf()￼￼toString()
propertyIsEnumerable()
hasOwnProperty()
```

#ARRAY! Prototype

```js
length
pop()
push()shift()
reverse()
sort()
join() 
reduce()slice()
```

#STRING! Prototype

```js
length
chartAt()
trim()concat()
indexOf()
replace()
toUpperCase()toLowerCase()￼￼￼￼￼￼￼￼substring()
```

#NUMBER! Prototype

```js
toFixed()
toExponential
toPrecision()
```

#FUNCTION! Prototype

```js
name
call()
bind()
apply()
```
[【javascript】call 和 apply](http://fireqqtw.logdown.com/posts/258035-javascriptcall-and-apply)


#Custom function in Prototype

```js
String.prototype.countAll = function ( letter ){ var letterCount = 0;for (var i = 0; i<this.length; i++) {        if ( this.charAt(i).toUpperCase() == letter.toUpperCase() ) {           letterCount++;        }}    return letterCount;};￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼
```

---

#build objects

```js
￼var myBox = { height: 6, width: 8, length: 10, volume: 480, material: "cardboard", contents: ["Great Expectations", "The Remains of the Day", "Peter Pan"]            };
```
```js
var shoe = { size: 6, gender: "women", construction: "slipper"};
￼var magicShoe = Object.create( shoe );￼￼￼￼console.log( magicShoe );
//=> Object { size: 6, gender: "women", construction: "slipper" }


￼Object.prototype.isPrototypeOf(show)
//=>true
shoe.isPrototypeOf(magicShoe)￼￼￼￼￼//=>true
```

constructor function for a shoe Object

```js
￼function Shoe (shoeSize, shoeColor, forGender, constructStyle) {   this.size = shoeSize;   this.color = shoeColor;   this.gender = forGender;   this.construction = constructStyle;}

Shoe.prototype = {  putOn: function () { alert ("Your " + this.construction + "’s" + "on!"); }, 
  takeOff: function () { alert ("Phew! Somebody’s size " + this.size + "’s" +" are fragrant! "); }
};

var beachShoe = new Shoe(10, "blue", "women", "flipflop");
```
---