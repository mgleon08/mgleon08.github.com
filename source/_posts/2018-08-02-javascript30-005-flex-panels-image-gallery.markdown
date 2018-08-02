---
layout: post
title: "Javascript30 - 005. Flex Panels Image Gallery"
date: 2018-08-02 22:44:33 +0800
comments: true
categories: javascript30
---

![](https://mgleon08.github.io/JavaScript30/005.Flex-Panels-Image-Gallery/images/thumbnail.png)

<!-- more -->

# [Demo](https://mgleon08.github.io/JavaScript30/005.Flex-Panels-Image-Gallery/index.html) | [Github](https://github.com/mgleon08/JavaScript30/tree/master/005.Flex-Panels-Image-Gallery)

練習 flex~

# box-sizing

元素的實際寬、高包含了內容的寬高加上padding 和 border長度

* `content-box` - 設定的寬度僅為內容寬度，而內距與邊框額外加上去。
* `border-box` - 設定的寬度就已經包含內容寬度、內距與邊框寬度。
* `inherit` - 繼承至父層的 broder-sizing 設定値。

```css
html {
 box-sizing: border-box;
}
```

* [CSS box-sizing 屬性筆記](https://mandywahahablog.wordpress.com/2015/05/20/css-box-sizing-%E5%B1%AC%E6%80%A7%E7%AD%86%E8%A8%98/)
* [關於 box-sizing 屬性](http://zh-tw.learnlayout.com/box-sizing.html)
* [CSS3 box-sizing 屬性](http://www.wibibi.com/info.php?tid=CSS3_box-sizing_%E5%B1%AC%E6%80%A7)

# overflow

* `visible` - 內容不會被修剪，當超出元素的範圍時內容會呈現在元素框之外。
* `hidden` - 內容會被修剪，但不會顯示捲軸，當超出元素的範圍時隱藏內容。
* `scroll` - 內容會被修剪，當超出範圍時自動變成捲軸呈現方式。
* `auto` - 自動選擇由瀏覽器決定如何顯示(預設值)，當超出範圍時自動變成捲軸呈現方式。
* `inherit` - 繼承至父層的 overflow 設定値。
* `overflow-x` - 可設定「水平」方向，當超出範圍時自動變成捲軸呈現方式。(需要內有寬度大於元素框的物件)
* `overflow-y` - 可設定「垂直」方向，當超出範圍時自動變成捲軸呈現方式。	

```css
.panels {
  min-height:100vh;
  overflow: hidden;
}
```

* [CSS overflow 內容「溢出邊界」區塊層元素](http://www.eion.com.tw/Blogger/?Pid=1158)

# transition 轉場

可以針對各種效果做轉場 `font-size` `flex` `background` ...

```css
.panel {
  /* Safari transitionend event.propertyName === flex */
  /* Chrome + FF transitionend event.propertyName === flex-grow */
  transition:
    font-size 0.7s cubic-bezier(0.61,-0.19, 0.7,-0.11),
    flex 0.7s cubic-bezier(0.61,-0.19, 0.7,-0.11),
    background 0.2s;
}
```
* [CSS 轉場](https://developer.mozilla.org/zh-TW/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions)

# e.propertyName

透過 `e.propertyName` 知道哪個 class 執行完 `transitionend` 觸發了 `toggleActive`，就可以針對想要判斷的做處理


```js
function toggleActive(e) {
  console.log(e.propertyName)
  // 因為在 Safari 為 flex, Chrome 為 flex-grow 因此用 includes
  if (e.propertyName.includes('flex')) {
    this.classList.toggle('open-active')
  }
}

panels.forEach(panel => panel.addEventListener('transitionend', toggleActive))
```
