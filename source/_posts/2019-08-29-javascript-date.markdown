---
layout: post
title: "javascript date"
date: 2019-08-29 00:31:13 +0800
comments: true
categories: javascript
---

<!-- more -->

在 javascript 中，時間是非常神奇的東西呢! 剛好遇到時區的相關問題，就來記錄一下

* GMT (格林威治標準時間, 格林威治平時, Greenwich Mean Time)
* UTC (協調世界時 Universal Time Coordinated)
* 一般情況下，GMT 和 UTC 可以互換，但是實際上，GMT 是一個時區，而 UTC 是一個時間標準

在 `js` 使用 `-` 會被 Chrome 解析為格林威治標準時間(GMT)的日期，所以像台灣的「本地時間」是 GMT+8 因此顯示時間會自動加上八小時來顯示。

若使用 `/` 則會是顯示「本地時間」

當想用 「本地時間」 時，請記得要改用 `/`

```js
// 調整電腦時區，顯示會不一樣

new Date('2019-08-14')
// Wed Aug 14 2019 08:00:00 GMT+0800 (台北標準時間)

new Date('2019/08/14')
// Wed Aug 14 2019 00:00:00 GMT+0800 (台北標準時間)
```

如果拿到的 date 是 `2019-08-14T00:00:00.000Z`，則可以透過轉換，來達成不被時區影響

```js
let date = '2019-08-14T00:00:00.000Z'
date = date.substring(0, 10).replace(/-/g, '/');
new Date(date);
```

在 [ECMAScript® Language Specification Date Time String Format](http://www.ecma-international.org/ecma-262/5.1/#sec-15.9.1.15) 有定義一套標準格式，基於 `ISO 8601`

```js
YYYY-MM-DDTHH:mm:ss
2019-08-14T00:00:00
```

也可以只有日期 (但就是會跟著時區跑)

```js
new Date('2019');
new Date('2019-08');
new Date('2019-08-14');
```

結論就是需要跟著時區用 `-`，不需要則用 `/`


# Reference

* [所有的時區](http://www.timeanddate.com/time/map/)
* [到底是 GMT+8 還是 UTC+8 ? - PanSci 泛科學](https://pansci.asia/archives/84978)
* [javascript - Why does Date.parse give incorrect results? - Stack Overflow](https://stackoverflow.com/questions/2587345/why-does-date-parse-give-incorrect-results)
* [Parse date without timezone javascript - Stack Overflow](https://stackoverflow.com/questions/17545708/parse-date-without-timezone-javascript)
* [ECMAScript Language Specification - ECMA-262 Edition 5.1](http://www.ecma-international.org/ecma-262/5.1/#sec-15.9.1.15)
* [前端工程研究：關於 JavaScript 中 Date 型別的常見地雷與建議作法 \| The Will Will Web](https://blog.miniasp.com/post/2016/09/25/JavaScript-Date-usage-in-details)
* [關於“時間”的一次探索 關於js時區iso，utc等完美解答](https://blog.csdn.net/XINGXUEXX/article/details/51132026)
