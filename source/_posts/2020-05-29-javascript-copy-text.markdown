---
layout: post
title: "Javascript - copy text"
date: 2020-05-29 00:17:52 +0800
comments: true
categories: javascript
---

<!-- more -->

點一下 button 就能夠複製的功能
 
```html
<!-- 一定要是 input 才能夠複製 --> 
<input type="hidden" id="copy-text" value="copy text">
```

```js
function copy() {
  // 選擇 dom 
  var copy = document.querySelector('#copy-text');
  // 將 dom 改成 text type
  copy.setAttribute('type', 'text');
  copy.select();
  // 執行複製
  document.execCommand('copy');
  // 再將原本的改成 hidden
  copy.setAttribute('type', 'hidden');
}
```
