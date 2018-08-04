---
layout: post
title: "Javascript30 - 007. Array Cardio Day 2"
date: 2018-08-04 16:18:27 +0800
comments: true
categories: javascript30
---

<!-- more -->

# [Demo](https://mgleon08.github.io/JavaScript30/007.Array-Cardio-Day-2/index.html) | [Github](https://github.com/mgleon08/JavaScript30/tree/master/007.Array-Cardio-Day-2)

練習 js 的 array 各種 methods 2

# some

check 只要有一個成立就 return true

```js
const isAdult = people.some(function(person) {
  const currentYear = (new Date()).getFullYear();
  if(currentYear - person.year >= 19) {
    return true;
  }
});
```

```js
const isAdult = people.some(person => ((new Date()).getFullYear()) - person.year >= 19);
```

# every

每個都要成立

```js
const allAdults = people.every(person => ((new Date()).getFullYear()) - person.year >= 19);
```

# find

第一個成立的就回傳

```js
const comment = comments.find(comment => comment.id === 823423);
```

# findIndex

第一個成立的 index

```js
const index = comments.findIndex(comment => comment.id === 823423);
```

# slice

```js
const newComments = [
  ...comments.slice(0, index),
  ...comments.slice(index + 1)
]
```

```js
var animals = ['ant', 'bison', 'camel', 'duck', 'elephant'];

console.log(animals.slice(2)) // 第二個到最後
// expected output: Array ["camel", "duck", "elephant"]

console.log(animals.slice(2, 4)) // 二到四(不包含四)
// expected output: Array ["camel", "duck"]

console.log(animals.slice(1, 5)) // 一到五(不包含五)
// expected output: Array ["bison", "camel", "duck", "elephant"]
```

* [slice](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)
