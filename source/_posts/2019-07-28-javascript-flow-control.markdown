---
layout: post
title: "JavaScript - Flow Control"
date: 2019-07-28 12:29:41 +0800
comments: true
categories: javascript
---

<!-- more -->

* [Booleans - Truthy & Falsy](#booleans)
* [Comparison Operators](#comparison)
* [if...else](#if)
* [switch](#switch)
* [loop](#loop)
* [邏輯運算子 Logical operators](#logical)
* [位元運算子 Bitwise operators](#bitwise)
* [Catching and Throwing Errors](#catch_error)

# <span id="booleans"> Booleans - Truthy & Falsy <span>

以下 6 個為 `Falsy` 其他都是 `Truthy `

```js
// Falsy
false
0, -0
""
null
undefined
NaN
```

```js
// Truthy
'Hello World'
8
[], [1, 2, 3]
{}, { name: 'leon' }
function foo() {}
true
...
```

可以用 `!!` 來做確認

```js
!!""
// false
!![]
// true
!!0
// false
```

* [Boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)

# <span id="comparison"> Comparison Operators <span>

* `===`：不做轉型，因此型別對比較是有影響的。
* `==`：會強制轉型
	* 布林轉數字
	* 字串轉數字
	* 使用 `valueOf()` 或 `toString()` 將物件取得基本型別的值，再做比較

```js
1 == 1.0          // true
'1' == 1          // true
'1' == 1.0        // true
null == undefined // true
NaN == NaN        // false
{} == {}          // false
[] == []          // false

1 === 1.0          // true
'1' === 1          // false
'1' === 1.0        // false
null === undefined // false
NaN === NaN        // false
{} === {}          // false
[] === []          // false

// reference same memory (object, array)
a = {}
b = a
a == b // true
a === b // true
```

```js
// 字串以字典的字母順序為主
'ab' < 'cd' // true
// 字串 '99' 被強制轉型為數字 99
'99' > 98 // true
// 字串 'Hello World' 無法轉為數字，變成 NaN
'Hello World' > 1 // false
```

* [Comparison operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators)

# <span id="if"> if...else <span>

```js
let age = 26
let isChild = age <= 7
let isSenior = age >= 65

if (isChild) {
  console.log('Welcome! You are free.')
} else if (isSenior) {
  console.log('Welcome! You get a discount.')
} else {
  console.log('Welcome!')
}
```

### 三元運算 Condition Ternary Operator

```js
function getFee(isMember) {
  return isMember ? '$2.00' : '$10.00'
}

console.log(getFee(true)) // "$2.00"
console.log(getFee(false)) // "$10.00"
console.log(getFee(1)) // "$2.00"
```

* [if...else](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else)
* [Conditional Operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)

# <span id="switch"> switch <span>

* `break` - 如果不加的話會繼續執行下去
* `default` - 如果上面都沒有執行，就會執行這行

```js
var expr = 'Papayas'
switch (expr) {
  case 'Oranges':
    console.log('Oranges are $0.59 a pound.')
    break
  case 'Mangoes':
  case 'Papayas':
    console.log('Mangoes and papayas are $2.79 a pound.')
    // expected output: "Mangoes and papayas are $2.79 a pound."
    break
  default:
    console.log('Sorry, we are out of ' + expr + '.')
}
```

* [switch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch)

# <span id="loop"> loop <span>

### for

```js
let str = ''

for (var i = 0; i < 9; i++) {
  str = str + i
}

console.log(str) // "012345678"
```

### for..in

```js
var string1 = ''
var object1 = { a: 1, b: 2, c: 3 }

for (var property1 in object1) {
  string1 += object1[property1]
}

console.log(string1) // "123"
```

### while


```js
var n = 0

while (n < 3) {
  n++
}

console.log(n) // 3
```

### do..while

```js
var result = ''
var i = 0

do {
  i = i + 1
  result = result + i
} while (i < 5)

console.log(result) // "12345"
```

* [for](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for)
* [for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in)
* [while](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/while)
* [do...while](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/do...while)

# <span id="logical"> 邏輯運算子 Logical operators <span>

```js
var a = 3
var b = -2

console.log(a > 0 && b > 0) // false
console.log(a > 0 || b > 0) // true
console.log(!(a > 0 || b > 0)) // false
```

* [Logical operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators)

# <span id="bitwise"> 位元運算子 Bitwise operators <span>

將運算元轉乘 32 位元的 0 和 1，`&` 兩個都是 1 就是 1，`|` 一個是 1 才是 1，最後在轉換成數字

```js
console.log(5 & 13) // 0101 & 1101 = 0101
// 5

console.log(parseInt('0101', 2) & parseInt('1101', 2))
// 5

console.log(5 & 13 & 3) // 0101 & 1101 & 0011 = 0001
// 1

console.log(5 | 13) // 0101 | 1101 = 1101
// 13
```

* [Bitwise operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators)

# <span id="catch_error"> Catching and Throwing Errors <span>

* `try..catch`: 捕獲 `error`，嘗試做其他處理
* `throw`: 拋出 `error`

```js
const getTip = amount => {
  if (typeof amount !== 'number') {
    throw Error('Argument must be a number')
  }
  return amount * 0.25
}

try {
  const result = getTip('12')
  console.log(result)
} catch (e) {
  console.log(e.message) // Argument must be a number
}
```

* [try...catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch)
