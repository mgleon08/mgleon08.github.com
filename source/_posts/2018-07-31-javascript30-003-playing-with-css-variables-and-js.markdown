---
layout: post
title: "Javascript30 - 003. Playing with CSS Variables and JS"
date: 2018-07-31 22:31:16 +0800
comments: true
categories: javascript30
---
<p><img src="https://mgleon08.github.io/JavaScript30/003.Playing-with-CSS-Variables-and-JS/images/thumbnail.png" alt="" /></p>

<!-- more -->

<h1 id="demohttpsmgleon08githubiojavascript30003playingwithcssvariablesandjsindexhtmlgithubhttpsgithubcommgleon08javascript30treemaster003playingwithcssvariablesandjs"><a href="https://mgleon08.github.io/JavaScript30/003.Playing-with-CSS-Variables-and-JS/index.html">Demo</a> | <a href="https://github.com/mgleon08/JavaScript30/tree/master/003.Playing-with-CSS-Variables-and-JS">Github</a></h1>

<p>利用 js 來更新 css 的效果!</p>

<h1 id="rangeinput">range input</h1>

<ul>
<li><code>data-sizing</code>: 自行新增屬性，用來判斷 size 的單位</li>
</ul>

<pre><code class="html language-html">&lt;label for="spacing"&gt;Spacing:&lt;/label&gt;
&lt;input type="range" name="spacing" min="10" max="200" value="10" data-sizing="px"&gt;
</code></pre>

<h1 id="dataset">dataset</h1>

<p>透過 js 抓取 <code>this.dataset</code>，就可以拿到自行新增屬性的值，回傳一個 object 回來，可以多個</p>

<pre><code class="js language-js">function handleUpdate() {
  this.dataset
}
</code></pre>

<p>也可以用 getAttribute</p>

<pre><code class="js language-js">function handleUpdate(){
  this.getAttribute('data-sizing')
}
</code></pre>

<h1 id="colorinput">color input</h1>

<p>在 browser 上產生，color 選擇器</p>

<pre><code class="html language-html">&lt;!-- value 一開始預設的值 --&gt;
&lt;label for="base"&gt;Base Color&lt;/label&gt;
&lt;input type="color" name="base" value="#ffc600"&gt;
</code></pre>

<h1 id="root">:root</h1>

<p>css 原生變數定義</p>

<pre><code class="css language-css">:root {
/* 定義變數的方法 */
  --base: #ffc600;
  --spacing: 10px;
  --blur: 10px;
}

img {
/* 變數的使用方式 */
  padding: var(--spacing);
}
</code></pre>

<ul>
<li><a href="http://muki.tw/tech/native-css-variables/">SASS, LESS 退散，原生 CSS 可以使用變數啦！</a></li>
</ul>

<h1 id="filter">filter</h1>

<p>css 的濾鏡效果</p>

<pre><code class="css language-css">img {
  filter: blur(10px); /* blur 模糊 */
}
</code></pre>

<ul>
<li><a href="https://developer.mozilla.org/zh-CN/docs/Web/CSS/filter">MDN filter</a></li>
</ul>

<h1 id="queryselectorall">querySelectorAll</h1>

<p>可抓取節點上的 dom，並且回傳值會是 <code>NodeList</code> 不是 <code>array</code></p>

<blockquote>
  <p>NodeList 不是 Array，但仍可以使用 forEach() 方法來進行迭代，但有些還是只有 Array 才會有</p>
</blockquote>

<pre><code class="js language-js">document.querySelectorAll('.controls input')
</code></pre>

<ul>
<li><a href="https://developer.mozilla.org/zh-TW/docs/Web/API/NodeList">MDN NodeList</a></li>

<li><a href="http://www.jstips.co/zh_tw/javascript/converting-a-node-list-to-an-array/">將 Node List 轉換成陣列</a></li>
</ul>

<h1 id="setproperty">setProperty</h1>

<p>更改屬性</p>

<pre><code class="js language-js">function handleUpdate() {
  const suffix = this.dataset.sizing || '';
  // 更改 style 的屬性，
  document.documentElement.style.setProperty(`--${this.name}`, this.value + suffix)
}
</code></pre>
