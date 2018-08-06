---
layout: post
title: "Javascript30 - 009. 14 Must Know Dev Tools Tricks"
date: 2018-08-06 23:03:31 +0800
comments: true
categories: javascript30
---

<!-- more -->

# [Demo](https://mgleon08.github.io/JavaScript30/009.14-Must-Know-Dev-Tools-Tricks/index.html) | [Github](https://github.com/mgleon08/JavaScript30/tree/master/009.14-Must-Know-Dev-Tools-Tricks)

14 個 debug 技巧

# 01. Regular

```js
console.log('hello')
```

# 02. Interpolated

```js
console.log('Hello I am a %s string!', '💩')
```

# 03. Styled

```js
console.log('%c I am some great text', 'font-size:50px background:red text-shadow: 10px 10px 0 blue')
```

# 04. warning!

```js
console.warn('OH NOOO')
```

# 05. Error :|

```js
console.error('Shit!')
```

# 06. Info

```js
console.info('Crocodiles eat 3-4 people per year')
```

# 07. Testing

```js
const p = document.querySelector('p')

console.assert(p.classList.contains('ouch'), 'That is wrong!')
```

# 08. clearing

```js
console.clear()
```

# 09. Viewing DOM Elements

```js
console.log(p)
console.dir(p)
```

# 10. Grouping together


```js
dogs.forEach(dog => {
  console.groupCollapsed(`${dog.name}`)
  console.log(`This is ${dog.name}`)
  console.log(`${dog.name} is ${dog.age} years old`)
  console.log(`${dog.name} is ${dog.age * 7} dog years old`)
  console.groupEnd(`${dog.name}`)
})
```


# 11. counting

```js
console.count('Wes')
console.count('Wes')
console.count('Steve')
console.count('Steve')
console.count('Wes')
console.count('Steve')
console.count('Wes')
console.count('Steve')
console.count('Steve')
console.count('Steve')
console.count('Steve')
console.count('Steve')
```

# 12. timing

```js
console.time('fetching data')
fetch('https://api.github.com/users/wesbos')
  .then(data => data.json())
  .then(data => {
    console.timeEnd('fetching data')
    console.log(data)
  })

console.table(dogs)
```

# 13. table

```js
console.table(dogs)
```
