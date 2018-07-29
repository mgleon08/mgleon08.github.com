---
layout: post
title: "Javascript30 - 001. JavaScript Drum Kit"
date: 2018-07-29 21:49:28 +0800
comments: true
categories: javascript30
---

![](https://mgleon08.github.io/JavaScript30/001.JavaScript-Drum-Kit/images/thumbnail.png)

<!-- more -->

# [Demo](https://mgleon08.github.io/JavaScript30/001.JavaScript-Drum-Kit/index.html) | [GitHub](https://github.com/mgleon08/JavaScript30/tree/master/001.JavaScript-Drum-Kit)
利用 js 監聽鍵盤 event，使其發出對應的音效

# window.addEventListener

監聽 event

```js
// 在全局監聽 keydown 動作，並執行 playSound function
window.addEventListener('keydown', playSound)

// 在所有 key 都加上監聽 transitionend (CSS Transition Events) 特效，特效結束後就執行 removeTransition function
keys.forEach(key => key.addEventListener('transitionend', removeTransition))

/*
相當於
keys.forEach(function (key) {
  return key.addEventListener('transitionend', removeTransition);
});
*/
```

# data-key

html 可以自行定義一個屬性為 `data-* attribute`

```html
<div class="key" data-key="65">
  <kbd>A</kbd>
  <span class="sound">clap</span>
</div>

<audio data-key="65" src="../sounds/clap.wav"></audio>
```

```js
// 透過 e.keyCode 拿到對應的 data-key
const key = document.querySelector(`.key[data-key="${e.keyCode}"]`)
const audio = document.querySelector(`audio[data-key="${e.keyCode}"]`)
```

* [[技術分享] 什麼是 HTML 5 中的資料屬性（data-* attribute）](https://pjchender.blogspot.com/2017/01/html-5-data-attribute.html)

# classList

在既有的 class 上面，新增或刪除 class

```js
key.classList.add('playing')
this.classList.remove('playing')
```

# audio

```html
<audio data-key="65" src="../sounds/clap.wav"></audio>
```

```js
// 設定時間
audio.currentTime = 0
// 開始播放
audio.play()
```

# vh

vh, vw 可視範圍，可隨著螢幕的長寬去做變化

```css
.keys {
  min-height: 100vh;
}
```

* [[筆記] 好用的css 3新單位vh vw ─ 讓你的圖片可以隨著螢幕大小而不同](https://pjchender.blogspot.com/2015/04/css-3vh-vw.html)

# shadow

```css
.key {
  text-shadow: 0px 0px 15px rgb(255, 251, 40); 
  /* 
    h-shadow：水平位移距離
    v-shadow：垂直位移距離
    blur：模糊半徑
    spread：擴散距離
    color：顏色 
  */
  
  box-shadow: 0 0 15px #ffc600;
  /* 
     h-shadow：水平位移距離
     v-shadow：垂直位移距離
     blur：模糊半徑
     spread：擴散距離
     color：顏色
     inset：內陰影
     最少要有 h-shadow 及 v-shadow 兩個參數
  */
}
```

* [[CSS3]text-shadow 文字陰影](https://abgne.tw/css/css3-lab/css3-text-shadow.html)
* [[CSS3]box-shadow 區塊陰影](https://abgne.tw/css/css3-lab/css3-box-shadow.html)

# letter-spacing

```css
.sound {
  font-size: 1.2rem;
  text-transform: uppercase; /* 轉大寫 */
  letter-spacing: 1px; /* 字間距離 */
  color: #ffc600;
}
```

# transform

控制元素的變形的效果 (translate 位移、scale 放大縮小、rotate 旋轉、skew 傾斜、[matrix](http://www.oxxostudio.tw/articles/201409/svg-20-transform-matrix.html))

```css
.playing {
  transform:scale(1.1); /* (x軸,y軸) 放大 1.1 倍 */
  border-color:#ffc600;
  box-shadow: 0 0 15px #ffc600;
}
```

* [CSS 2D Transforms](https://www.w3schools.com/css/css3_2dtransforms.asp)
* [SVG 研究之路 (19) - transform 基礎篇](http://www.oxxostudio.tw/articles/201409/svg-19-transform.html)

參考文章

* [Wes Bos javascript30](https://javascript30.com/)
* [「JS30紀錄＆心得」01 - JavaScript Drum Kit](https://guahsu.io/2017/05/JavaScript30-01-Java-Script-Drum-Kit/)
