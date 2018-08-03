---
layout: post
title: "Javascript30 - 006. Ajax Type Ahead"
date: 2018-08-03 23:57:24 +0800
comments: true
categories: javascript30
---

![](https://mgleon08.github.io/JavaScript30/006.Ajax-Type-Ahead/images/thumbnail.png)

<!-- more -->

# [Demo](https://mgleon08.github.io/JavaScript30/006.Ajax-Type-Ahead/index.html) | [Github](https://github.com/mgleon08/JavaScript30/tree/master/006.Ajax-Type-Ahead)

練習 ajax

# outline

繪製於元素周圍的一條線，位於邊框邊緣的外圍，可起到突出元素的作用，用法跟 border 一樣

```css
p {
  outline: #4CAF50 solid 10px;
}
```

* [CSS Outline](https://www.w3schools.com/css/css_outline.asp)

# text-transform

可改變字體的大小寫

* `none`	No capitalization. The text renders as it is. This is default	
* `capitalize`	Transforms the first character of each word to uppercase
* `uppercase`	Transforms all characters to uppercase
* `lowercase`	Transforms all characters to lowercase
* `initial`	Sets this property to its default value. Read about initial	
* `inherit`	Inherits this property from its parent element. Read about inherit	

```css
a {
    text-transform: uppercase;
}
```

* [CSS text-transform Property](https://www.w3schools.com/cssref/pr_text_text-transform.asp)


# nth-child

可以針對每個元素做個別的樣式設定

```css
/* 針對偶數 */
.suggestions li:nth-child(even) {
  transform: perspective(100px) rotateX(3deg) translateY(2px) scale(1.001);
  background: linear-gradient(to bottom,  #ffffff 0%,#EFEFEF 100%);
}

/* 針對奇數 */
.suggestions li:nth-child(odd) {
  transform: perspective(100px) rotateX(-3deg) translateY(3px);
  background: linear-gradient(to top,  #ffffff 0%,#EFEFEF 100%);
}
```

* [使用 CSS nth-child 必須要注意的事情](http://muki.tw/tech/css-nth-child-notice/)

# linear-gradient

顏色漸層

```css
/* 漸變軸為45度，從藍色漸變到紅色 */
linear-gradient(45deg, blue, red);

/* 從右下到左上、從藍色漸變到紅色 */
linear-gradient(to left top, blue, red);

/* 從下到上，從藍色開始漸變、到高度40%位置是綠色漸變開始、最後以紅色結束 */
linear-gradient(0deg, blue, green 40%, red);
```

* [MDN linear-gradient](https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient)


# fetch

透過 fetch 拿取 endpoint 資料

```js
// fetch 會回傳一個包含 response 的 promise 
fetch(endpoint)
  .then(blob => blob.json()) // 透過 fetch 拿取 json 資料
  .then(data => cities.push(...data));
```

* [MDN fetch](https://developer.mozilla.org/zh-TW/docs/Web/API/Fetch_API/Using_Fetch)


# 展開運算子 ...

可將 array 展開來，也算是淺拷貝的一種

```js
fetch(endpoint)
  .then(blob => blob.json())
  .then(data => cities.push(...data));
```

```js
const params = [ "hello", true, 7 ]
const other = [ 1, 2, ...params ] // [ 1, 2, "hello", true, 7 ]
```

* [展開運算子與其餘運算子](https://eyesofkids.gitbooks.io/javascript-start-from-es6/content/part4/rest_spread.html)

# RegExp & match

* `RegExp` 用來做正規表達式
* `match` mapping 字串

```js
  function findMatches(wordToMatch, cities) {
    return cities.filter(place => {
      // here we need to figure out if the city or state matches what was searched
      // i 不分大小寫，g 全域搜尋 global search
      const regex = new RegExp(wordToMatch, 'gi')
      return place.city.match(regex) || place.state.match(regex)
    })
  }
```

* [MDN Regular_Expressions](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Guide/Regular_Expressions)


# replace

將字串替換掉

```js
const regex = new RegExp(this.value, 'gi')
// 若找到符合 this.value e.g. bos 就替換成 <span class="hl">bos</span>
const cityName = place.city.replace(regex, `<span class="hl">${this.value}</span>`)
```

```js
function numberWithCommas(x) {
  return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
}
```

* [MDN Regular_Expressions](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Guide/Regular_Expressions)
