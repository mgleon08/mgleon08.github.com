---
layout: post
title: "javascript 時區(timezone) & 夏令時間(Daylight saving time)"
date: 2019-08-29 00:31:13 +0800
comments: true
categories: javascript
---

<!-- more -->

前端的時間會根據 GMT 時間，減掉後再帶回後端，因此當前端選擇用 `/` 去 parse 時間，會出現時間不會加上時區的時間，因此在減掉時區的時間後，反而帶到後端的時間會變成前一天

### new Date()

```js
// 前端
2020-04-17T03:11:29.686Z

// 後端
Fri Apr 17 2020 11:11:29 GMT+0800 (台北標準時間)
```

### new Date('1979-08-01')

```js
// 前端
Wed Aug 01 1979 09:00:00 GMT+0900 (台北夏令時間)

// 後端
1971-08-01T00:00:00.000Z
```

### new Date('1979/08/01')

```js
// 前端
Wed Aug 01 1979 00:00:00 GMT+0900 (台北夏令時間)

// 後端
1979-07-31T15:00:00.000Z
```

可以看到只有 `new Date('1979/08/01')` 帶到後端的時候變成 `1979-07-31T15:00:00.000Z`，後端存日期就會錯誤，因此如果前端用  `/` ，必須將 timezoneOffset 加回來

可以使用套件

* [Day.js](https://day.js.org/en/)

```js
dayjs.utc(date, 'YYYY/MM/DD').add(dayjs().utcOffset(), 'minute').toDate()
```

* [momentjs.com](https://momentjs.com/)

```js
moment(date).hours(new Date().getTimezoneOffset()/-60)._d;
```

# 夏令時間

有發現上面的 `1979-08-01` 後面的時間會是 `+9`?? 台灣不是 `+8` 嗎?? 

點連結發現其實台灣以前也有夏令時間，因此只要日期是那個時間內，就會是 `+9`，所以記得 `utcOffset()` and `getTimezoneOffset` 記得要用當下的時間去取得

```js
var d = new Date('1979/08/01')
var offset = ((new Date().getTimezoneOffset())/-60)
var now = moment(d).hours(offset)._d;

Wed Aug 01 1979 00:00:00 GMT+0900 (台北夏令時間)
 8
Wed Aug 01 1979 08:00:00 GMT+0900 (台北夏令時間)
// 08:00 但 GMT 是 +0900，所以帶回後端，-9 就會變成 1979/7/31
```

所以正確應該是要

* [Day.js](https://day.js.org/en/)

```js
// 帶 date 進去
dayjs.utc(date, 'YYYY/MM/DD').add(dayjs(date).utcOffset(), 'minute').toDate()
```

* [momentjs.com](https://momentjs.com/)

```js
// 帶 date 進去
moment(date).hours(new Date(date).getTimezoneOffset()/-60)._d;

// 當遇到有時候會有時區有夏令時間時，可改用 minutes，因為用 hours 會因為有些 hour 會是 7.5 hours 後變成 7 導致有一小時的差距
moment(date).minutes(new Date().getTimezoneOffset()/-1)._d;
```

或是後端吃 string 的話

```js
new Date(date).toDateString()
```

或是最好就是用標準時間

```js
YYYY-MM-DDTHH:mm:ss
2019-08-14T00:00:00
```

# Reference

* [夏令時間](http://www.freehoro.net/Name/Tutorial/DayLightSavingTime/ZhongHua.html)
* [消失的神祕１小時！美國夏令日光節約（Daylight saving time）](https://solomo.xinmedia.com/travel/15197-Daylightsavingtime)
* <a href="{{ root_url }}/blog/2019/08/29/javascript-date/"> Javascript Date </a>
2020-04-17-javascript-daylight-saving-time.markdown
2020-04-17-javascript-daylight-saving-time.markdown
