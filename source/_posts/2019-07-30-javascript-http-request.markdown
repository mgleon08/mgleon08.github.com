---
layout: post
title: "JavaScript - HTTP request, Promise, async/await"
date: 2019-07-30 12:32:23 +0800
comments: true
categories: javascript
---

<!-- more -->

* [XMLHttpRequest](#XMLHttpRequest)
* [Promise](#promise)
* [Async/Await](#async_await)

Asynchronous(異步/非同步) vs. Synchronous(同步)

* `Asynchronous`: 不需等待，可以繼續做別的事
* `Synchronous`: 必須等待事情完成，才能繼續做別的事

# <span id="XMLHttpRequest"> XMLHttpRequest </span>

```js
const request = new XMLHttpRequest()
request.addEventListener('readystatechange', e => {
  // 4 代表已 request 完成，已經有 response
  if (e.target.readyState === 4) {
    const data = JSON.parse(e.target.responseText)
    console.log(data)
  }
})
request.open('GET', 'http://puzzle.mead.io/puzzle')
request.send()
```

透過 `callback function` 可以當 response 回來時，在執行

```js
const getPuzzle = callback => {
  const request = new XMLHttpRequest()
  request.addEventListener('readystatechange', e => {
    if (e.target.readyState === 4 && e.target.status === 200) {
      const data = JSON.parse(e.target.responseText)
      callback(undefined, data.puzzle)
    } else if (e.target.readyState === 4) {
      callback('An error has taken place', undefined)
    }
  })
  request.open('GET', 'http://puzzle.mead.io/puzzle?wordCount=3')
  request.send()
}


// callback function
getPuzzle((error, puzzle) => {
  if (error) {
    console.log(`Error: ${error}`)
  } else {
    console.log(puzzle)
  }
})
```

* [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
* [Callback function](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function)
* [非同步(Asynchronous)與同步(Synchronous)的差異](https://medium.com/@hyWang/%E9%9D%9E%E5%90%8C%E6%AD%A5-asynchronous-%E8%88%87%E5%90%8C%E6%AD%A5-synchronous-%E7%9A%84%E5%B7%AE%E7%95%B0-c7f99b9a298a)

# <span id="promise"> Promise </span>

簡單的來說，`Promise` 主要用來處理非同步狀態(async), 就是承諾當任務完成時，通知下一個任務可以開始，不會像上面一樣要一直寫 `callback` 導致一些 `callback hell`

Promise 的生命週期

* `pending` - 等待中的初始狀態
* `fulfillment` - 完成
* `rejecttion` - 失敗

```js
// Callback
const getDataCallBack = callback => {
  setTimeout(() => {
    callback(undefined, 'This is the callback data')
  }, 2000)
}

getDataCallBack((err, data) => {
  if (err) {
  } else {
    console.log(data)
  }
})

// Promise
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    // 成功做什麼事
    resolve('This is the promise resolve')
    // 失敗做什麼事
    reject('This is the promise reject')
  }, 2000)
})

myPromise.then(
  data => {
    console.log(data)
  },
  err => {
    console.log(err)
  }
)
```

### Promise chain

用 `catch` 取代 error handler `err => {}`

```js
const getDataPromise = num =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      typeof num === 'number' ? resolve(num * 2) : reject('Number must be provided')
    }, 2000)
  })

getDataPromise(10)
  .then(data => {
    getDataPromise(data)
      .then(data => {
        console.log(data) // 40
      })
      .catch(err => {
        console.log(err)
      })
  })
  .catch(err => {
    console.log(err)
  })

// better way
getDataPromise(10)
  .then(data => {
    return getDataPromise(data)
  })
  .then(data => {
    console.log(data) // 40
  })
  .catch(err => {
    console.log(err)
  })
```

也有一些非同步模式的變體可以使用

* `Promise.all`: 等所有都完成再進行下一步
* `Promise.race`: 只要一個先完成就進行下一步

### fetch

`fetch` 會回傳 `promise object`，因此也可以用 `then` 去接

```js
const fetch = require('node-fetch')

fetch('http://puzzle.mead.io/puzzle', {})
  .then(response => {
    if (response.status === 200) {
      return response.json()
    } else {
      throw new Error('Unable to fetch the puzzle')
    }
  })
  .then(data => {
    console.log(data.puzzle)
  })
  .catch(error => {
    console.log(error)
  })
```

* [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
* [JavaScript Promises: an Introduction](https://developers.google.com/web/fundamentals/primers/promises)
* [Promise — 介紹與使用](https://medium.com/@xyz030206/promise-%E4%BB%8B%E7%B4%B9%E8%88%87%E4%BD%BF%E7%94%A8-66605ef56e34)
* [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
* [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)
* [你懂 JavaScript 嗎？#24 Promise](https://cythilya.github.io/2018/10/31/promise/)

# <span id="async_await"> Async/Await </span>

`async function` 也會回傳 Promise 的函式

* `async`: 定義一個 `function` 為 `async`
* `await`: 等待某一 `function` return 後再繼續執行，必須包在 `async` 裡面

```js
const test = async () => {}
test() // Promise {<resolved>: undefined}
```

```js
const processData = async () => {
  // throw new Error('error')
  return 'hi'
}

processData()
  .then(data => {
    console.log(data)
  })
  .catch(error => {
    console.log(error)
  })
```

```js
// node need to require
const fetch = require('node-fetch')

const getPuzzle = async wordCount => {
  const response = await fetch(`http://puzzle.mead.io/puzzle?wordCount=${wordCount}`)
  if (response.status === 200) {
    const data = await response.json()
    return data.puzzle
  } else {
    throw new Error('Unable to get puzzle')
  }
}
getPuzzle('2')
  .then(puzzle => {
    console.log(puzzle)
  })
  .catch(err => {
    console.log(`Error: ${err}`)
  })
```

* [async](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
* [await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
* [我要學會 JS(三)：callback、Promise 和 async/await 那些事兒](https://noob.tw/js-async/)
