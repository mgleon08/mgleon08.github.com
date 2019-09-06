---
layout: post
title: "JavaScript - Functions, Object, Classes, this, Closures, Array"
date: 2019-07-29 12:31:12 +0800
comments: true
categories: javascript
---

<!-- more -->

* [Default function parameters](#default)
* [立即函示 IIFE(Immediately Invoked Function Expression)](#IIFE)
* [Arrow Function](#arrow)
* [this 識別字（this Identifier)](#this)
* [Closures 閉包](#closures)
* [Object](#object)
* [Methods](#methods)
* [Prototype Object](#prototype)
* [Array](#array)
* [Constructor](#constructor)
* [Classes](#classes)

# <span id="default"> Default function parameters </span>

```js
let getScoreText = function (name = 'Anonymous', score = 0) {
  return `${name} Score: ${score}`
}
let text = getScoreText(undefined, 11)
console.log(text) // Anonymous Score: 11
```

* [Default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)

# <span id="IIFE"> 立即函示 IIFE(Immediately Invoked Function Expression) </span>

可立刻執行的函示，好處不會污染到 global，在函示後面加上 `()` 即可

```js
(function() {
  var a = 1
  console.log(a) // 3
})()

a // a is not defined
```

# <span id="arrow"> Arrow Function <span>

ES6 新增的寫法

```js
const squareLong1 = function(num) {
	return num * num
}
console.log(squareLong1(3))

const squareLong2 = (num) => {
	return num * num
}
console.log(squareLong2(3))

const squareLong3 = (num) => num * num
console.log(squareLong3(3))
```

Arrow Function don't bind `arguments`

```js
const add1 = function() {
  return arguments[0] + arguments[1]
}

console.log(add1(1, 2)) // 3

const add2 = () => {
  return arguments[0] + arguments[1]
}

console.log(add2(1, 2))

// [object Object]function require(path) {
//     try {
//       exports.requireDepth += 1;
//       return mod.require(path);
//     } finally {
//       exports.requireDepth -= 1;
//     }
//   }
```

Arrow Function don't bind `this`

因為 `arrow function` 沒有綁定 `this` 因此會 `undefined`

```js
// arrow function
const pet1 = {
  name: 'Hal',
  getGreeting: () => {
    return `Hello ${this.name}!`
  }
}
console.log(pet1.getGreeting()) // Hello undefined!

// 原本寫法
const pet2 = {
  name: 'Hal',
  getGreeting: function() {
    return `Hello ${this.name}!`
  }
}
console.log(pet2.getGreeting()) // Hello Hal!

// 原本寫法縮寫
const pet3 = {
  name: 'Hal',
  getGreeting() {
    return `Hello ${this.name}!`
  }
}
console.log(pet3.getGreeting()) // Hello Hal!
```

```js
var test = {
  say: 'hi',
  citys: ['a', 'b', 'c'],
  getGreeting: function() {
    this.citys.forEach(function(city) {
    	// 這裡的 this 會指向 window
      console.log(this.say);
    });
  }
};

test.getGreeting(); // undefined

var test = {
  say: 'hi',
  citys: ['a', 'b', 'c'],
  getGreeting: function() {
    that = this;
    this.citys.forEach(function(city) {
      console.log(that.say);
    });
  }
};

test.getGreeting(); // hi, hi, hi

var test = {
  say: 'hi',
  citys: ['a', 'b', 'c'],
  getGreeting: function() {
    this.citys.forEach(city => console.log(this.say));
  }
};

test.getGreeting(); // hi, hi, hi

var test = {
  say: 'hi',
  citys: ['a', 'b', 'c'],
  getGreeting() {
    this.citys.forEach(city => console.log(this.say));
  }
};
test.getGreeting(); // hi, hi, hi
```

* [Arrow_functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
* [Shorter_functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions#Shorter_functions)
* [ES6 Method definitions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions#Description)

# <span id="constructor"> Constructor <span>

constructor 的用法就是 `function` 搭配 `new` 關鍵字：

可以透過 `Constructor function` 達成 `Object-oriented programming (OOP)`，就像是 `class` 那樣

```js
const Person = function(firstName, lastName, age) {
  this.firstName = firstName
  this.lastName = lastName
  this.age = age
}

const me = new Person('Leon', 'Ji', 27)
// me.[[prototype]] = Person.prototype 實際上會做此連結 hidden internal property
console.log(me) // Person {firstName: "Leon", lastName: "Ji", age: 27}
```

* 建議使用大寫開頭
* 透過 `new` 建立新的 instance

### new 會進行以下操作

* 建立一個物件（`{}`)
* 將這個新物件的 constructor 屬性設為另一個物件(`Person`)

```js
me.constructor == Person // true
me instanceof Person // true
```

* 讓新物件繼承 `Person.prototype`

```js
Person.prototype.say = 'hi'
me.say // hi
```

* 如果該函式沒有返回物件，則返回 this

這裡的 `this` 就是 `Person`

* [new](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new)
* [Using a constructor function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects#Using_a_constructor_function)
* [筆記 Javascript 的 constructors, prototypes, new 關鍵字](https://andyyou.github.io/2016/09/22/js-contrstructors-prototypes-new/)

# <span id="classes"> Classes <span>

後來出的新語法 `class`，讓原本的 `function` 寫法更乾淨，更像 OOP 一點

```js
class Person {
  constructor(firstName, lastName, age, likes = []) {
    this.firstName = firstName
    this.lastName = lastName
    this.age = age
    this.likes = likes
  }

  getBio() {
    let bio = `${this.firstName} is ${this.age}.`
    this.likes.forEach(like => {
      bio += ` ${this.firstName} likes ${like}.`
    })
    return bio
  }

  setName(fullName) {
    const name = fullName.split(' ')
    this.firstName = name[0]
    this.lastName = name[1]
  }
}

const me = new Person('Leon', 'Ji', 27, ['Teaching', 'Biking'])
console.log(me.getBio())
const person2 = new Person('Clancey', 'Turner', 51)
person2.setName('Leon Ji')
console.log(person2.getBio())

// Leon is 27. Leon likes Teaching. Leon likes Biking.
// Leon is 51.
```

### Subclass

可以在建立一個 class，去繼承其他的 class

```js
// stu -> Student.prototype -> Person.prototype -> Object.prototype -> null

class Student extends Person {
  constructor(firstName, lastName, age, grade, likes) {
    // super 用於 call 上層的 constructor
    super(firstName, lastName, age, likes)
    this.grade = grade
  }
  updateGrade(change) {
    this.grade += change
  }
  getBio() {
    const status = this.grade >= 70 ? 'passing' : 'failing'
    return `${this.firstName} is ${status} the class.`
  }
}

const stu = new Student('Leon', 'Ji', 27, 50, ['Teaching', 'Biking'])
console.log(stu.getBio())
student.updateGrade(50)
console.log(stu.getBio())

// Leon is failing the class.
// Leon is passing the class.
```

### getter & setter

* The `get` syntax binds an object property to a function that will be called when that property is looked up.
* The `set` syntax binds an object property to a function to be called when there is an attempt to set that property.

執行時，就跟變數一樣，不需加上 `()`

```js
const human = {
  firstName: 'Leon',
  lastName: 'Ji',
  get name() {
    return `${this.firstName} ${this.lastName}`
  },
  set name(name) {
    const names = name.trim().split(' ')
    this.firstName = names[0]
    this.lastName = names[1]
  }
}
human.name = ' Leon Ji '
console.log(human.firstName) // Leon
console.log(human.lastName) // Ji
```

* [Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)
* [extends](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends)
* [getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get)
* [setter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set)


# <span id="this"> this 識別字（this Identifier) <span>

this 是 function 執行時所屬的物件，而 this 是在執行時期做繫結，其值和函式在哪裡被呼叫（call-site）有關。

```js
function foo() {
  console.log(this.bar)
}

var bar = 'global'

var obj1 = {
  bar: 'obj1',
  foo: foo
}

var obj2 = {
  bar: 'obj2'
}

foo() // 'global'
obj1.foo() // 'obj1'
foo.call(obj2) // 'obj2'
new foo() // undefined
```

匹配的優先順序由高至低排列

1. new 繫結：`this` 會指向 `new` 出來的物件。
	* `new foo()` sets `this` to a brand new empty object.
2. 明確繫結：使用 `call`、`apply`、`bind`，明確指出要繫結給 `this` 的物件。
	* `foo.call(obj2)` sets `this` to the `obj2` object.
3. 隱含繫結：當函式為物件的方法（method）時，在執行階段 `this` 就會被繫結至該物件。
	* `obj1.foo()` sets `this` to the `obj1` object.
4. 預設繫結：當其他規則都不適用時，意即沒有使用 `call`、`apply`、`bind` 或不屬於任何物件的 method，就套用預設繫結，在非嚴格模式下，瀏覽器環境 `this` 的值就是預設值全域物件 window，而在嚴格模式下，`this` 的值就是 `undefined`。


### 重點是誰呼叫它

```js
function callName() {
  console.log(this.name);
}
var name = 'global leon';
var say = {
  name: 'leon',
  callName: callName  
  // 這裡的 function 指向全域的 function，但不重要
}
callName()     // global leon
say.callName() // leon
```

### 透過 bind 綁定正確的 context

```js
const obj = {
  name: 'foobar',
  getName() {
    return this.name;
  }
};

console.log(obj.getName()); // foobar
const getName = obj.getName;
console.log(getName()); // error
const getNameWithBind = obj.getName.bind(obj);
console.log(getNameWithBind()); // foobar
```


* [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
* [你懂 JavaScript 嗎？#3 暖身 (๑•̀ㅂ•́)و✧ Part 2 - 變數、嚴格模式、IIFEs、閉包、模組、this、原型、Polyfill 與 Transpiler](https://cythilya.github.io/2018/10/10/intro-2/)
* [this-identifier](https://github.com/getify/You-Dont-Know-JS/blob/master/up%20%26%20going/ch2.md#this-identifier)
* [鐵人賽：JavaScript 的 this 到底是誰？](https://wcc723.github.io/javascript/2017/12/12/javascript-this/)

# <span id="closures"> Closures 閉包 <span>

> A closure is the combination of a function and the lexical environment within which that function was declared.

閉包 (Closure) 是一種特殊的函式，他能夠存取被宣告當下的環境中的變數。

```js
const createCounter = () => {
  let count = 0
  return () => {
    return count
  }
}
const counter = createCounter()
console.log(counter()) // 0
```

```js
// return 一個 function，baseTip 已被存取
const createTipper = baseTip => {
  return amount => {
    return baseTip * amount
  }
}

const tip20 = createTipper(0.2)
const tip30 = createTipper(0.3)
console.log(tip20(100)) // 20
console.log(tip20(80))  // 16
console.log(tip30(100)) // 30
```

```js
var a = 100
function echo() {
  console.log(a)
}

function test() {
  var a = 200
  echo()
}

test() // 100
```

* [Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
* [[教學] 史上最簡單！3分鐘看懂JavaScript閉包 (Closure)](http://shubo.io/javascript-closure/)
* [所有的函式都是閉包：談 JS 中的作用域與 Closure](https://blog.techbridge.cc/2018/12/08/javascript-closure/)

# <span id="object"> Object <span>

```js
let myBook = {
    title: '1984',
    author: 'George Orwell',
    pageCount: 326
}

console.log(`${myBook.title} by ${myBook.author}`)
myBook.title = 'Leon'
console.log(myBook)
```

* [Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)

# Object Reference

output 出相同的值，因為 function 裡的 object 是 reference 到同一個 memory，

```js
let myAccount = {
    name: 'Leon',
    expenses: 0,
    income: 0
}
let addExpense = function (account, amount) {
  account.expenses = account.expenses + amount
  console.log(account) // {name: "Leon", expenses: 2000, income: 0}
}
addExpense(myAccount, 2000)
console.log(myAccount) // {name: "Leon", expenses: 2000, income: 0}
```

但是當 assign 一個新的 object 給 account，就會是不一樣的 object

```js
// Both logs print differen things
let myAccount = {
    name: 'Leon',
    expenses: 0,
    income: 0
}
let addExpense = function (account, amount) {
    account = {}
    account.age = 1
    console.log(account) // {age: 1}
}

addExpense(myAccount, 2000)
console.log(myAccount) // {name: "Leon", expenses: 0, income: 0}
```

# <span id="methods"> Methods <span>

```js
let restaurant = {
  name: 'ASB',
  guestCapacity: 75,
  guestCount: 0,
  checkAvailability: function(partySize) {
    let seatsLeft = this.guestCapacity - this.guestCount
    return partySize <= seatsLeft
  }
}
console.log(restaurant.checkAvailability(4))
```

String methods

```js
let name = 'Leon'
console.log(name.length)
console.log(name.toUpperCase())
console.log(name.toLowerCase())

let password = 'abc123asdf098'
console.log(password.includes('password'))
```

Number & Math methods

```js
let num = 103.941
console.log(num.toFixed(2))

console.log(Math.round(num))
console.log(Math.floor(num))
console.log(Math.ceil(num))

let min = 0
let max = 1
let randomNum = Math.floor(Math.random() * (max - min + 1)) + min
```

* [String methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)
* [Number methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)
* [Math methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math)

# <span id="prototype"> Prototype Object <span>

Prototype 可說是物件的一種 fallback 機制，當在此物件找不到指定屬性時，就會透過原型鏈結（prototype link / prototype reference）追溯到其父物件上。

透過 `prototype` 可以 shared 給每個 `instance`，因為 `instance` 都是繼承自 `prototype property`

以下 `new` 出來的 instance 都繼承自 `Person prototype`

```js
const Person = function(firstName, lastName, age, likes = []) {
  this.firstName = firstName
  this.lastName = lastName
  this.age = age
  this.likes = likes
}

Person.prototype.getBio = function() {
  let bio = `${this.firstName} is ${this.age}.`
  this.likes.forEach(like => {
    bio += ` ${this.firstName} likes ${like}.`
  })

  return bio
}

Person.prototype.setName = function(fullName) {
  const name = fullName.split(' ')
  this.firstName = name[0]
  this.lastName = name[1]
}

const me = new Person('Leon', 'Ji', 27, ['Teaching', 'Biking'])
// me.[[prototype]] = Person.prototype 實際上會做此連結 hidden internal property
console.log(me.getBio())
const person2 = new Person('Clancey', 'Turner', 51)
person2.setName('Leon Ji')
console.log(person2.getBio())
```

所有 object 都有 `hasOwnProperty` 是因爲都繼承自 `Object.prototype`

```js
// myObject --> Object.prototype --> null
const myObject = {} console.log(myObject.hasOwnProperty('doesNotExist'))

// 因為 myObject 本身並沒有 hasOwnProperty，而是繼承而來
myObject.hasOwnProperty('hasOwnProperty') // false
// __proto__ 上一層
myObject.__proto__.hasOwnProperty('hasOwnProperty') // true
```

基本型別 Primitives values 的 prototype

`string`, `number`, `boolean`, `undefined`, `null`

```js
// Array: myArray --> Array.prototype --> Object.prototype --> null
// Function: myFunc --> Function.prototype --> Object.prototype --> null
// String: myString --> String.prototype --> Object.prototype --> null
// Number: myNumber --> Number.prototype --> Object.prototype --> null
// Boolean: myBoolean --> Boolean.prototype --> Object.prototype --> null
```

* [Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
* [Object prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype)
* [Array.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/prototype)
* [Function.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/prototype)
* [String.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/prototype)
* [Number.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/prototype)
* [Boolean.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean/prototype)


# <span id="array"> Array <span>

```js
const nums = [1]
nums.push(12)
nums.unshift(3)
console.log(nums) // [3, 1, 12]

nums.pop()
console.log(nums) // [3, 1]

nums.shift()
console.log(nums) // [1]

nums.splice(1, 0, 2)
console.log(nums) // [1, 2]
nums.splice(1, 0, 3)
console.log(nums) // [1, 3, 2]
nums.splice(1, 2, 99)
console.log(nums) // [1, 99]
```

### forEach

```js
const todos = ['Order cat food', 'Clean kitchen', 'Buy food', 'Do work', 'Exercise']

todos.forEach(function (todo, index) {
	const num = index + 1
	console.log(`${num}. ${todo}`)
})

for (let num = 1; num <= todos.length ; num++) {
  console.log(`${num}. ${todos[num]}`)
}
```

### indexOf

```js
const places = ['a', 'b', 'c']
const index = places.indexOf('c')
console.log(index) // 2
const index2 = places.indexOf('d')
console.log(index2) // -1

// 無法用在 object 因為 {} === {} false，兩個 object 會是不同的 memory
const testObject = [{}]
console.log(testObject.indexOf({})) // -1
```

### findIndex

for object should use `findIndex`

```js
const notes = [
  {
    title: 'My next trip',
    body: 'I would like to go to Spain'
  },
  {
    title: 'Habbits to work on',
    body: 'Exercise. Eating a bit better.'
  },
  {
    title: 'Office modification',
    body: 'Get a new seat'
  }
]
const index = notes.findIndex(function(note, index) {
  return note.title === 'Habbits to work on'
})
console.log(index) // 1
```

### find

```js
const notes = [
  {
    title: 'My next trip',
    body: 'I would like to go to Spain'
  },
  {
    title: 'Habbits to work on',
    body: 'Exercise. Eating a bit better.'
  },
  {
    title: 'Office modification',
    body: 'Get a new seat'
  }
]
const findNote = function(notes, noteTitle) {
  return notes.find(function(note, index) {
    return note.title.toLowerCase() === noteTitle.toLowerCase()
  })
}
const note = findNote(notes, 'my next trip')
console.log(note) // Will print the first object from our array above
```

### filter

```js
var words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];

const result = words.filter(word => word.length > 6);

console.log(result);
// expected output: Array ["exuberant", "destruction", "present"]
```

```js
const todos = [
  {
    text: 'Order cat food',
    completed: false
  },
  {
    text: 'Clean kitchen',
    completed: true
  },
  {
    text: 'Do work',
    completed: false
  }
]
const getThingsToDo = function(todos) {
  return todos.filter(function(todo) {
    return !todo.completed
  })
}
console.log(getThingsToDo(todos))
```

### sorting

```js
var months = ['March', 'Jan', 'Feb', 'Dec']
months.sort()
console.log(months)
// expected output: Array ["Dec", "Feb", "Jan", "March"]

var array1 = [1, 30, 4, 21, 100000]
array1.sort()
console.log(array1)
// expected output: Array [1, 100000, 21, 30, 4]
```

```js
const todos = [
  {
    text: 'Buy food',
    completed: true
  },
  {
    text: 'Do work',
    completed: false
  },
  {
    text: 'Exercise',
    completed: true
  }
]
const sortTodos = function(todos) {
  todos.sort(function(a, b) {
    if (!a.completed && b.completed) {
      return -1
    } else if (!b.completed && a.completed) {
      return 1
    } else {
      return 0
    }
  })
}
sortTodos(todos)
console.log(todos)
```

### Rest Parameter

```js
const calculateAverage = (...numbers) => {
  let sum = 0
  numbers.forEach(num => (sum += num))
  return sum / numbers.length
}
console.log(calculateAverage(0, 100, 88, 64)) // Will print: 63
```

```js
const printTeam = (teamName, coach, ...players) => {
  console.log(`Team: ${teamName}`)
  console.log(`Coach: ${coach}`)
  console.log(`Players: ${players.join(', ')}`)
}
printTeam('Liberty', 'Casey Penn', 'Marge', 'Aiden', 'Herbert', 'Sherry')
```

### Spread Syntax

```js
const printTeam = (teamName, coach, ...players) => {
  console.log(`Team: ${teamName}`)
  console.log(`Coach: ${coach}`)
  console.log(`Players: ${players.join(', ')}`)
}
const team = {
  name: 'Libery',
  coach: 'Casey Penn',
  players: ['Marge', 'Aiden', 'Herbert', 'Sherry']
}
printTeam(team.name, team.coach, ...team.players)
```

```js
let cities = ['Barcelona', 'Cape Town', 'Bordeaux']
let citiesClone = [...cities, 'Santiago']
console.log(cities)
console.log(citiesClone)

// [ 'Barcelona', 'Cape Town', 'Bordeaux' ]
// [ 'Barcelona', 'Cape Town', 'Bordeaux', 'Santiago' ]
```

* [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
* [Rest parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)
* [Spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)
