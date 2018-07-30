---
layout: post
title: "Javascript30 - 002. CSS + JS Clock"
date: 2018-07-30 23:10:35 +0800
comments: true
categories: javascript30
---

![](https://mgleon08.github.io/JavaScript30/002.CSS+JS-Clock/images/thumbnail.png)

<!-- more -->

# [Demo](https://mgleon08.github.io/JavaScript30/002.CSS+JS-Clock/index.html) | [Github](https://github.com/mgleon08/JavaScript30/tree/master/002.CSS%2BJS-Clock)

利用 js + css 效果做出時鐘

# translateY

y 軸方向的距離 (translateX, translateZ, translate3d..s)

```css
.clock-face {
  position: relative;
  width: 100%;
  height: 100%;
  transform: translateY(-3px); /* account for the height of the clock hands */
}
```

* [MDN translateY()](https://developer.mozilla.org/en-US/docs/Web/CSS/transform-function/translateY)
* [transform:translate()（元素的表示位置進行平移）](http://webkkl.com/style/translate.php)


# transform-origin

旋轉元素的基點位置(default: 50%)

```css
.hand {
  transform-origin: 100%; /* 100% 最右邊 */
}
```

* [CSS沒有極限 - CSS transform-origin](https://wcc723.github.io/css/2013/10/10/css-transform-origin/)

# transition-timing-function

> 做出時鐘頓點的效果

數率曲線

* ease：`cubic-bezier(0.25, 0.1, 0.25, 1)`
* ease-in: `cubic-bezier(0.42, 0,1, 1)`
* ease-out: `cubic-bezier(0, 0, 0.58, 1)`
* ease-in-out: `cubic-bezier(0.42, 0, 0.58, 1)`
* linear: `cubic-bezier(0, 0, 1, 1)`

```css
.hand {
  transition-timing-function: cubic-bezier(0.1, 2.7, 0.58, 1);
}
```

* [CSS transition 各種速率](https://wcc723.github.io/css/2013/08/24/css-transtion-speed/)
* [timing-functions](http://devdocs.io/css/timing-function#The_cubic-bezier()_class_of_timing-functions)
* [實用的 CSS — 貝塞爾曲線(cubic-bezier)](https://segmentfault.com/a/1190000004618375)

# setInterval

定時器，可以設定多久就執行指定的動作

```js
// setInterval(function, ms time)
setInterval(setDate, 1000)
```

* [[javascript] 深入瞭解 setTimeout() 與 setInterval() 的不同之處](https://blog.camel2243.com/2016/08/06/javascript-%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3-settimeout-%E8%88%87-setinterval-%E7%9A%84%E4%B8%8D%E5%90%8C%E4%B9%8B%E8%99%95/)

# Date Time

透過 js 時間函示，拿到時, 分, 秒，在轉換成度數，呈現在 `rotate(${度數}deg)`

```js
const now = new Date()
const hours = now.getHours()
const mins = now.getMinutes()
const secs = now.getSeconds()
```
