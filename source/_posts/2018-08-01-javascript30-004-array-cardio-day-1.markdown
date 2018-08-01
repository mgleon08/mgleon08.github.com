---
layout: post
title: "Javascript30 - 004. Array Cardio Day 1"
date: 2018-08-01 22:53:10 +0800
comments: true
categories: javascript30
---

<!-- more -->

# [Demo](https://mgleon08.github.io/JavaScript30/004.Array-Cardio-Day-1/index.html) | [Github](https://github.com/mgleon08/JavaScript30/tree/master/004.Array-Cardio-Day-1)

練習 js 的 array 各種 methods

# console.table

在 console 將 data 以 table 的形式呈現

```js
console.table(data)
```

# filter

```js
const fifteen = inventors.filter(function(inventor){
  if(inventor.year >= 1500 && inventor.year <= 1600){
    return true
  }
})
```

```js
const fifteen = inventors.filter(inventor => (inventor.year >= 1500 && inventor.year <= 1600))
```

# map

```js
const fullname = inventors.map(inventor => (inventor.first + ' ' + inventor.last))
```

```js
const fullname = inventors.map(inventor => `${inventor.first} ${inventor.last}`)
```

# sort

```js
const ordered = inventors.sort(function(a, b){
  if(a.year > b.year){
    return 1
  }else{
    return -1
  }
})
```

```js
const ordered = inventors.sort((a, b) => a.year > b.year ? 1 : -1)
```

# reduce


```js
var totalYears = 0
for(var i = 0; i < inventors.length; i++){
  totalYears += (inventors[i].passed - inventors[i].year)
}
```

```js
const totalYears = inventors.reduce((total, inventor) => {
  return total + (inventor.passed - inventor.year)
}, 0)
```

# combo

[https://en.wikipedia.org/wiki/Category:Boulevards_in_Paris](https://en.wikipedia.org/wiki/Category:Boulevards_in_Paris)

```js
const category = document.querySelector('.mw-category')
const links = Array.from(category.querySelectorAll('a'))
const de = links.map(link => link.textContent).filter(streetName => streetName.includes('de'));
```

```js
const links = [...category.querySelectorAll('a')]
```

> category.querySelectorAll 會是 NodeList 因此要用 `Array.from` or `[...]` 轉成 array 才能夠使用 map
