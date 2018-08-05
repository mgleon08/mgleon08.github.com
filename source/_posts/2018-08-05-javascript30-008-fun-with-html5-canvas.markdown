---
layout: post
title: "Javascript30 - 008. Fun with HTML5 Canvas"
date: 2018-08-05 17:10:09 +0800
comments: true
categories: javascript30
---

![](https://mgleon08.github.io/JavaScript30/008.Fun-with-HTML5-Canvas/images/thumbnail.png)

<!-- more -->

# [Demo](https://mgleon08.github.io/JavaScript30/008.Fun-with-HTML5-Canvas/index.html) | [Github](https://github.com/mgleon08/JavaScript30/tree/master/008.Fun-with-HTML5-Canvas)

透過 HTML 的canvas 搭配 Javascript 做出畫布的效果。

> 要加 ; 否則會有 error

# canvas

透過 `canvas.getContext` 定義為 2d 繪圖

```js
const canvas = document.querySelector('#draw');
const ctx = canvas.getContext('2d');
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;
```

* `ctx.strokeStyle` 設定或返回用於筆觸的顏色、漸變或模式
* `ctx.lineJoin` 設定或返回兩條線相交時，所建立的拐角類型
* `ctx.lineCap` 設定或返回線條的結束端點樣式
* `ctx.lineWidth` 設定或返回當前的線條寬度


```js
ctx.strokeStyle = '#BADA55';
ctx.lineJoin = 'round';
ctx.lineCap = 'round';
ctx.lineWidth = 100;
```

* `ctx.beginPath()` 起始一條路徑，或重置當前路徑
* `ctx.moveTo(a,b)` 把路徑行動到畫布中的指定點，不建立線條
* `ctx.lineTo(a,b)` 新增一個新點，然後在畫布中建立從該點到最後指定點的線條
* `ctx.stroke()` 繪製已定義的路徑

```js
ctx.beginPath(); // start from
ctx.moveTo(lastX, lastY); // go to
ctx.lineTo(e.offsetX, e.offsetY);
ctx.stroke();
[lastX, lastY] = [e.offsetX, e.offsetY];
```

# EventListener

* `mousedown` 按下滑鼠
* `mousemove` 移動滑鼠
* `mouseup`   放開滑鼠
* `mouseout` 滑鼠移開視窗

```js
canvas.addEventListener('mousedown', (e) => {
  isDrawing = true;
  [lastX, lastY] = [e.offsetX, e.offsetY];
});
canvas.addEventListener('mousemove', draw);
canvas.addEventListener('mouseup', () => isDrawing = false);
canvas.addEventListener('mouseout', () => isDrawing = false);
```

* [MDN Canvas 基本用途](https://developer.mozilla.org/zh-TW/docs/Web/API/Canvas_API/Tutorial/Basic_usage)
* [HTML 5 Canvas 參考手冊](http://www.w3school.com.cn/tags/html_ref_canvas.asp)
