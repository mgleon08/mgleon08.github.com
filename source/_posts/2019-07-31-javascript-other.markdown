---
layout: post
title: "JavaScript - localStorage, sessionStorage, location, window, Date, Destructuring, Module"
date: 2019-07-31 12:32:53 +0800
comments: true
categories: javascript
---

<!-- more-->

* [localStorage](#localStorage)
* [sessionStorage](#sessionStorage)
* [location](#location)
* [Window](#window)
* [Date](#date)
* [解構賦值 Destructuring](#destructuring)
* [Module](#module)

# <span id="localStorage"> localStorage </span>

* 優點：存放不會消失，不需擔心頁面關閉/重整/新開分頁等問題
* 缺點：因為永久存放(除非user手動清除)，必須加上時效性自行判斷，或特殊操作時要將其手動清除

`localStorage` 只能存 string

```js
localStorage.setItem('foo', 'bar')
console.log(localStorage.getItem('foo'))
localStorage.removeItem('foo')
localStorage.clear() // delete all
```

如果要存 `array`, `object` 必須先轉成 `string`

```js
userJSON = JSON.stringify(user)
localStorage.setItem('user', userJSON)

userString = localStorage.getItem('user')
userParse = JSON.parse(userString)
console.log(`${userParse.name} and ${userParse.age}`)
```

* [localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)

# <span id="sessionStorage"> sessionStorage </span>

* 優點：頁籤開著時會一直存在，重新整理也不會消失，頁籤關閉時會自動銷毀被儲存的資訊
* 缺點：只存在於當前頁籤，開新分頁並不會被傳遞

```js
// Save data to sessionStorage
sessionStorage.setItem('key', 'value');

// Get saved data from sessionStorage
let data = sessionStorage.getItem('key');

// Remove saved data from sessionStorage
sessionStorage.removeItem('key');

// Remove all saved data from sessionStorage
sessionStorage.clear();
```

* [sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)


# <span id="location"> location </span>

The Location interface represents the location (URL) of the object it is linked to

```js
// 轉址到 google
location.assign('http://www.google.com/')

// 取得網址後面參數
location.hash.substring(1)
```

* [Location](https://developer.mozilla.org/en-US/docs/Web/API/Location)


# <span id="window"> Window </span>

The Window interface represents a window containing a DOM document

```js
// 跨頁面同步資料, 透過 listen 'storage' 是否改變去做同步
window.addEventListener('storage', function (e) {
// Will fire for localStorage changes that come from a different page
})
```

* [window](https://developer.mozilla.org/en-US/docs/Web/API/Window)


# <span id="date"> Date </span>

```js
const dateOne = new Date('March 1 2017 12:00:00')
const month = dateOne.getMonth() + 1
const day = dateOne.getDate()
const year = dateOne.getFullYear()
console.log(`${month}/${day}/${year}`)
```

```js
const date = new Date('March 1 2017 12:00:00')
const timestamp = date.getTime()
console.log(timestamp) // Will print 1488387600000
```

```js
const timestamp = 1488387600000
const date = new Date(timestamp)
```

* [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
* [momentjs](https://momentjs.com/)

# <span id="destructuring"> 解構賦值 Destructuring </span>

透過 mapping 屬性名稱，給予賦值

In Object

```js
const todo = {
  id: 'ididididididid',
  text: 'Pay the bills',
  completed: false
}
const { text: todoText, completed, details = 'No details provided', ...others } = todo
console.log(todoText) // "Pay the bills"
console.log(completed) // false
console.log(details) // "No details provided"
console.log(others) // { id: "ididididididid" }
```

In function arguments

```js
const todo = {
  id: 'asdfpoijwermasdf',
  text: 'Pay the bills',
  completed: false
}
const printTodo = ({ text, completed }) => {
  console.log(`${text}: ${completed}`)
}
printTodo(todo)
```

In Array

```js
const age = [65, 0, 13]
const [firstAge, ...otherAges] = age
console.log(firstAge) // 65
console.log(otherAges) // [0, 13]
```

`Object` 解構是以名稱做對應，`Array` 則是以順序為對應

```js
let [a, , c] = [1, 2, 3]
console.log(a, c) // 1, 3

let {a, ,c} = {a:1, b:2, c:3}
console.log(a,c) // Error
```

解構用於提取 json 值時，非常方便

```js
let foo = {
  a: 1,
  b: 'bar',
  c: {
    d: 'hi',
    e: 'hello'
  }
}

let { a, b, c } = foo
console.log(a, b, c)
// 1 'bar' { d: 'hi', e: 'hello' }
```

* [Destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
* [[筆記] JavaScript ES6 中的物件解構賦值（object destructuring）](https://pjchender.blogspot.com/2017/01/es6-object-destructuring.html)

# <span id="module"> Module </span>

### Named export

```js
// utilities.js
export const add = (a, b) => a + b
export const subtract = (a, b) => a - b

// index.js
import { add, subtract } from './utilities'
console.log(add(32, 1)) // 33
console.log(subtract(32, 1)) //  31
```

### default export

```js
// utilities.js
const add = (a, b) => a + b
const subtract = (a, b) => a - b
const square = (x) => x * x
export { add, subtract, square as default }
```

```js
// index.js
import otherSquare, { add } from './utilities'
console.log(add(32, 1)) // 33
console.log(otherSquare(10)) // 100
```

* [import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)
* [export](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export)
