---
layout: post
title: "JavaScript - Variables"
date: 2019-07-27 12:28:47 +0800
comments: true
categories: javascript
---

<!-- more -->

* [命名規則](#naming)
* [宣告 var、let、const](#declare)
* [Variable Scope](#scope)
* [undefine and null](#undefine_null)
* [Type Coercion](#type_coercion)
* [嚴格模式 Strict Mode](#strict)

# <span id="naming"> 命名規則 <span>

> A JavaScript identifier must start with a letter, underscore (_), or dollar sign ($) subsequent characters can also be digits (0-9). Because JavaScript is case sensitive, letters include the characters "A" through "Z" (uppercase) and the characters "a" through "z" (lowercase).

From [MDN Variables](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types#Variables)

### 關鍵字 keyword & 保留字 reserved word

在變數命名時，要注意不要跟這些字一樣

* [keyword](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#Reserved_keywords_as_of_ECMAScript_2015) - 指在目前 ECMAScript 中有特定用途的英文字詞
* [Future reserved keywords](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#Future_reserved_keywords) - 系統留用，雖然目前尚未用到但未來可能有其他用途的字彙。

# <span id="declare"> 宣告 var、let、const <span>


* `var`
	* 函式作用域(`function scope`)
	* 可重複宣告
	* 可重新賦值
* `let` (ES6)
	* 區塊作用域(`block scope`)
	* 不可重複宣告
	* 可重新賦值
* `const` (ES6)
	* 區塊作用域(`block scope`)是
	* 不可重複宣告
	* 不可重新賦值
  * `const` 主要用於常數，像是圓面積的 `π` `const PI = 3.14`

因此 `var` 在 `function` 不會被外部讀取到，但在 `if`, `else`, `for`, `while` 等等區塊語句宣告時，則會變成 `global variable`，導致一些無法預期的事發生，如果希望都以區塊當範圍，則改用 `let`

### 重複宣告

當使用 `let` 和 `const` 時，不可以重複宣告一樣的變數名稱 (`var` 可以覆蓋，所以建議改用 `let` 和 `const`)

```js
var name = 'hi'
var name = 'hello'
console.log(name) // hello
```

```js
let name = 'hi'
let name = 'hello'

// SyntaxError: Identifier 'name' has already been declared
console.log(name)
```

## Example 1

```js
function test() {
  var a = 1
  let b = 2
  if (true) {
    var a = 3 // scope 是 function，因此可以改變 a = 1 的值
    let b = 4 // scope 是 block，只存在於 if(){}
  }
  console.log(a) // 3
  console.log(b) // 2
}

test()
```

## Example 2

```js
for (i = 0; i < 5; ++i) {
  setTimeout(function() {
    console.log(i)
  }, 1000 * i)
}
// 5 5 5 5 5
```

原因在於 `var` 宣告後，會變成 `global variable` 而變成以下

```js
var i
for (i = 0; i < 5; ++i) {
  setTimeout(function() {
    console.log(i)
  }, 1000 * i)
}
```

解法改用 `let` 或是 `IIFE 立即函示 (Immediately Invoked Function Expression)`

```js
// let
for (let i = 0; i < 5; ++i) {
  setTimeout(function() {
    console.log(i)
  }, 1000 * i)
}
// 0 1 2 3 4
```
```js
// IIFE
for (var i = 0; i < 5; ++i) {
  setTimeout((function(j) {
    return function() {
      console.log(j)
    }
  })(i), 1000 * i)
}
// 0 1 2 3 4
```
* [Declarations](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements#Declarations)

# String Variable

```js
let city = 'Taipei'
let country = 'Taiwan'
let myLocation = country + ' ' + city
```

### Template literals

使用 \` 包起來，裡面的變數則用 `${}`

```js
let template_location = `${country} ${city}`
```

* [string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)

# Numbers Variable

```js
let number = 1 + .3
let number2 = 1 - -2
let number3 = 1 * 2
let number4 = 1 / 2

let studentScore = 18
let maxScore = 20
let percent = (studentScore / maxScore) * 100
```

* [Arithmetic operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators)
* [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)

# String + Number

兩邊的值資料型別不同，當其中一方是字串時，`+` 所代表的就是字串運算子，而將數字強制轉型為字串

```js
let string = '100'
let number = 100
let total = string + number // "100100"
```

透過 `Number()` 先將字串轉為數字即可

```js
let string = '100'
let number = 100
let total = Number(string) + number // 200
```

# <span id="scope"> Variable Scope <span>

## Global Scope vs Local Scope

* `Global Scope 全域變數` - 在 `function scope(var)`和 `block scope(let, const)` 之外宣告的變數，全域變數在整個程式中都可以被存取與修改。
* `Local Scope 區域變數` - 在 `function scope(var)` 和 `block scope(let, const)` 內宣告，每次執行函式時，就會建立區域變數再予以摧毀，而且函式之外的所有程式碼都不能存取這個變數。

```js
// global scope
let varOne = 1

if (true) {
  // local scope, block scope
  let varTwo = 2
}

function test(){
	// local scope, function scope
	let varThree
}
```

## variable shadowing

指在某變數可視範圍內，定義了同名變數，在後者的可視範圍中，取用同一名稱時所用的是後者的定義。

```js
// global scope
let x = 1

if (x === 1) {
  // local scope
  let x = 2
  console.log(x) // 2
}

console.log(x) // 1
```

## Leaked global

當沒有用 `var` `let` `const` 宣告時，要取得 global variable 時，找不到會自動 create 一個

```js
// Example 1: Leaked global
let leakedGlobal = function() {
  if (true) {
    score = 3
  }
  console.log(score)
}
leakedGlobal()
console.log(score) // 3
```

```js
// Example 2: No leaked global
let leakedGlobal = function() {
  let score
  if (true) {
    score = 3
  }
  console.log(score)
}
leakedGlobal()
console.log(score) // Will throw an error as score wasn't leaked to the global scope
```

## Hoisting 拉升

> 在程式執行前，編譯器（compiler）會先由上到下逐行將程式碼轉為電腦可懂的命令，然後再執行編譯後的指令。在這個編譯的階段，編譯器找出所有的變數並繫結所屬範疇，但不賦值，所以此刻變數所帶的值是 undefined；而在執行階段，JavaScript 引擎才會處理給值的事情。

```js
var x = 8

function hoisting1(){
  console.log(x)
  var x = 5
}

hoisting1() // undefined
```

相當於底下，x 編譯時先被提升到上面

```js
var x = 8

function hoisting2(){
  var x // 編譯時期的工作
  console.log(x)
  x = 5 // 執行時期的工作
}

hoisting2() // undefined
```

改成用 `let` 就會出現 error

```js
var x = 8

function hoisting3(){
  console.log(x)
  let x = 5
}

hoisting3() // Cannot access 'x' before initialization
```

# <span id="undefine_null"> undefine and null <span>

* `undefine` - 當宣告變數但沒給值，`default argument`，或是沒 return 值時，就會是 `undefine`，也可以像一般值一樣 assign

```js
let test1
console.log(test1) // undefined

function foo(num){
	if(num === undefined){
		console.log(num) // undefined
	}
}
test2 = foo()

test3 = undefined // undefined 也可以 assign
```

* `null` - 可以像一般值一樣 assign，或是像 `match` 沒有和匹配時，就會是 `null`

```js
let name = 'leon'
name = null
if (name === null) {
  console.log('No name is set!')
}
```

```js
function getVowels(str) {
  var m = str.match(/[aeiou]/gi);
  if (m === null) {
    return 0;
  }
  return m.length;
}

console.log(getVowels('sky'));
```

* [undefined](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/undefined)
* [null](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/null)

# <span id="type_coercion">  Type Coercion <span>

typeof

```js
console.log(typeof 43)            // number
console.log(typeof NaN)           // number
console.log(typeof 'Andrew')      // string
console.log(typeof undefined)     // undefined
console.log(typeof function() {}) // function
console.log(typeof null)          // object
```

因為 js 會強制轉型，不會 error 導致會發生一些無法預期的行為

```js
const value = false + 12
const type = typeof value
console.log(type)  // "number"
console.log(value) // 0

// false 會轉成 0
// true 會轉成 1
```

* [typeof](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof)
* [Coercion](https://github.com/getify/You-Dont-Know-JS/blob/master/types%20%26%20grammar/ch4.md)

# <span id="strict"> 嚴格模式 Strict Mode <span>

預防開發者的一些不小心或錯誤的行為，JavaScript 引擎協助做了一些檢測的工作(防止像 `Leaked global`)

未宣告變數而賦值的狀況下，會無預警的產生一個全域變數，使用 Strict Mode 則會禁止

```js
'use strict'

a = 1 // Uncaught ReferenceError: a is not defined
```

```js
'use strict'
// let data // 加上這行就可以了
const processData = () => {
    data = '1230987234'
}
processData()
console.log(data) // Uncaught ReferenceError: data is not defined
```

* [Strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)
* [ECMA-262-5 in detail. Chapter 2. Strict Mode.](http://dmitrysoshnikov.com/ecmascript/es5-chapter-2-strict-mode/)
