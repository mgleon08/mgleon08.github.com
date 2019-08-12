---
layout: post
title: "Functional Programming 函式程式設計"
date: 2019-07-26 14:57:55 +0800
comments: true
categories: design_pattern
---

<!-- more -->

之前一直沒有特別研究什麼事 `Function Programming`，這次就好好的來研究一下

# 介紹

`函式式程式設計 Functional Programming (FP)` 是 `程式設計 programming paradigm` 的其中一種，是一類典型的程式設計風格，其他像是 `物件導向程式設計 Object-oriented programming (OOP)` 也是其中一種。

不同的程式設計，有不同的設計概念，因此用來解決問題的`思考`方式也大不相同，所以必須先瞭解這些語言的理念，才有辦法對症下藥。

必須符合以下的一項或多項重要概念(最重要的屬於前兩項)

> 函式就算定義為「高階函式」，也不一定就能稱為「函式程式設計」，符合函式程式設計有一定的要件，你還必須確保該函式要能「避免改變狀態」、「避免可變的資料」以及擁有「純函式」等特性。

# 1. First-class 一等公民

對待函式(Function) 如同對待其他資料型別一樣，例如可以像變數一樣 `直接賦予值` 或是 `當作參數傳遞` (`javascript` 就屬於此特性)

* 直接賦予值

```js
// variable
var a = 1
const a = 1

// function
var a = function(a,b) { return a+b; }
add(1, 2); // 3
```

# 2. higher-order functions 高階函式

至少會滿足下列其中一項條件

* 可以將函式(至少一個)當成參數傳入另一個函式

```js
var add = function (a, b) { return a+b; }
var calc = function (op, a, b) { return op(a, b) };

// add 傳入 calc
calc(add, 1, 2); // 3
```

* 可以將函式當成另一個函式的回傳值

```js
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var addOne = add(1);
addOne(2); // 3
```

# 3. pure functions 純函式

意指將相同的傳入 (Input)丟入函式，永遠會回傳相同的 output 結果，而且在過程中完全沒有任何的副作用。

意指函式在執行的時候，完全相同的參數傳入 (Input)，永遠會回傳相同的傳出 (Output)，而且在過程中`完全沒有任何副作用`，且函式程式設計更加強調「執行的結果」而非「執行的過程」，倡導利用幾個簡單的函式來計算結果，

副作用包括像是

* 狀態改變
* 資料改變
* 更改外部變數或者物件屬性(例如：全域變數、父類別範圍內的變數等)
* 寫入console.log、檔案
* 觸發外部流程
* 呼叫任何有副作用的函式(Functions)
* ...


```js
// slice: Pure Function
var arr = [1, 2, 3, 4, 5, 6];
arr.slice(0, 3); // output = [1, 2, 3], arr = [1, 2, 3, 4, 5, 6]
arr.slice(0, 3); // output = [1, 2, 3], arr = [1, 2, 3, 4, 5, 6]

// splice: not Pure Function
var arr = [1, 2, 3, 4, 5, 6];
arr.splice(0, 3); // output = [1, 2, 3], arr = [4, 5, 6]
arr.splice(0, 3); // output = [4, 5, 6], arr = []
```

可以發現到，`slice()` 每次 output 都會是一樣，而 `splice()` 則會不同

# Declarative vs Imperative

`函式程式設計 Functional Programming` 屬於 `宣告式程式設計 Declarative Paradigm` 的一種

* 指令式程式設計(Imperative Paradigm)
	* 程式碼具體表達需要做什麼來達到目標。描述該做什麼(how to do)以及流程控制(flow control)。
	* ex: C、JAVA

```js
var words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];
var result = [];

for(i=0; i<words.length; i++){
	word = words[i];
	if (word.length > 6){
		result.push(word);
	}
}

console.log(result);
// expected output: Array ["exuberant", "destruction", "present"]
```

* 宣告式程式設計(Declarative Paradigm)
	* 較為抽象的程式碼，可以藉由自然語言直觀的理解該行程式碼想要達到什麼樣的結果。描述該在哪做什麼(what to do)以及資料流程(data flow)。
	* ex: HTML、SQL

```js
var words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];

const result = words.filter(word => word.length > 6);

console.log(result);
// expected output: Array ["exuberant", "destruction", "present"]
```

js 裡的 `fliter` `map` 和 `reduce` 屬於高階函式

* [Array.prototype.filter()
](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
* [Array.prototype.map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
* [Array.prototype.reduce()
](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)

# 結語

之前接觸的大多是 `Object-oriented programming (OOP)` 透過這次更加了解 `Functional Programming (FP)`，不然之前都以為 `javascript` == `Functional Programming`，原來指的不是語言，而是設計風格啊~

# Reference

* [Functional programming Wiki](https://en.wikipedia.org/wiki/Functional_programming)
* [JavaScript: Functional Programming 函式程式設計概念](https://medium.com/@totoroLiu/javascript-functional-programming-%E5%87%BD%E5%BC%8F%E7%B7%A8%E7%A8%8B%E6%A6%82%E5%BF%B5-e8f4e778fc08)
* [前端工程研究：理解函式程式設計核心概念與如何進行 JavaScript 函式程式設計](https://blog.miniasp.com/post/2016/12/10/Functional-Programming-in-JavaScript)
* [Functional Programming in Javascript 教學網頁](http://reactivex.io/learnrx/)
