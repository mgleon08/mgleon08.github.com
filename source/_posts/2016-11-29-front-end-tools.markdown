---
layout: post
title: "前端工具整理 NVM,NPM,YARN,Webpack,Babel 等等"
date: 2016-11-29 11:32:26 +0800
comments: true
categories: javascript
---

前端工具，名詞實在太多了，這篇就來簡單記錄一下!

<!-- more -->

* [NVM](#nvm)
* [NPM](#npm)
* [YARN](#yarn)
* [Bower](#bower)
* [Grunt](#grunt)
* [Gulp](#gulp)
* [Yeoman](#yeoman)
* [Browserify](#browserify)
* [Webpack](#webpack)
* [Babel](#babel)

#<span id="nvm">NVM</span>

>管理 npm 工具，類似 rvm

```ruby
#根據當下最新版本
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.4/install.sh | bash

#將以下放到自己的 ~/.zshrc or ~/.bashrc or .bash_profile 下面（預設會自動放好，但還是去確定一下）
export NVM_DIR="/Users/leon/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm%

#重新載入 Shell
. ~/.nvm/nvm.sh or source ~/.zshrc

# 安裝穩定版本的 NodeJS
nvm install stable

#顯示目前可以安裝的版本
nvm ls-remote

#安裝 NodeJS
nvm install <version>

#安裝穩定版本的 NodeJS
nvm install stable

#使用版本，只有在當下，重新開新tab就會消失
nvm use stable

#設定預設版本，永久
nvm alias default stable

#看目前安裝所有版本
ls -a ~/.nvm/versions/node
```
* [node版本管理工具nvm-Mac下安装及使用](https://segmentfault.com/a/1190000004404505)
* [在 Mac 上裝 NVM, RVM 等跑 Gulp, Grunt, Bower 環境](https://tenten.co/blog/install-gulp-grunt-bower-sass-susy-on-mac-with-nvm-rvm/)
* [Node.js 安裝與版本切換教學 (for MAC)](http://icarus4.logdown.com/posts/175092-nodejs-installation-guide)

#<span id="npm">NPM</span>

>套件管理

[npm](https://www.npmjs.com/)

資料夾一定會有 package.json

```ruby
#搜尋 npm 套件，但建議去網站上比較快
npm search

#本地安裝，會安裝在當前專案的 node_modules 目錄下
npm install <package name>

#全域安裝，會將套件安裝在統一的 npm 目錄底下
npm install -g <package name>

#列出專案使用套件
npm ls (-g 全域套件)

#更新專案套件
npm update (-g 全域套件)

#移除專案套件
npm uninstall <package name> (-g 全域套件)

#清快取
npm cache clean

#查詢 npm 儲存路徑
npm config get prefix

#自動安裝 package.json 套件定義檔中定義的所有套件
npm install

#安裝套件並儲存在 package.json 中
npm install <package name> --save #用於上線時必要的套件(react, bootstrap…)，會更新到package.json裡的Dependencies(上線依賴)
npm install <package name> --save-dev #用來安裝開發時用的工具(ex babel, webpack, webpack-dev-server…)，會更新到package.json裡的devDependencies(開發依賴)
```

###package.json

```ruby
#自動產生 package.json
npm init

{
  "name": "leon",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies":{#套件相依
  },
  "devDependencies": {#開發套件相依
    "lodash": "^4.15.0" #示範加上去的
  }
}
```

* [Npm 套件管理 & 常用開發工具介紹](http://www.slideshare.net/wantingj/npm-46801372)


#<span id="yarn">Yarn</span>

Facebook 開源的 Yarn，這是針對存儲在 npm 或 Bower 註冊表中的 JavaScript 模組的一個代理包管理器。

```ruby
brew install yarn

# -g 為 global 的意思，沒有加的話，會裝在當下的
yarn global add vue-cli

# 安裝指定套件 (會自動 save 到 package.json)
yarn add sass sass-loader node-sass

# 安裝 package.json 內其它套件
yarn install

# run project
yarn run start
```

* [yarn](https://yarnpkg.com/en/)
* [yarn github](https://github.com/yarnpkg/yarn)
* [yarn migrating from npm](https://yarnpkg.com/en/docs/migrating-from-npm)
* [[譯] Yarn 官方介紹: 一款新的 JavaScript 包管理器](https://github.com/cssmagic/blog/issues/67)
* [Facebook 新發佈的 Yarn JS 包管理器的 5 大功能](https://sheerdevelopment.com/posts/facebook-js-5)

#<span id="bower">Bower</span>

[Bower](https://bower.io/search/) 由 Twitter 團隊開發的前端套件管理工具，用來管理或安裝 Web 開發所需要的 Package，像是 CSS 和 JavaScript，也可以依據套件的相依性來安裝

> 簡單來說，開發者不用再去煩惱套件相依性問題

> 主要用來做前端資源依賴管理，跟npm很像，區別在於：npm管理的是node模組的依賴，bower管理的是前端資源的依賴，如css、javascript文件等。

* 快速管理與安裝網頁前端套件。
* 易於檢視專案的套件相依性，僅需檢查 bower.json 即可知道專案使用了哪些套件及版本。
* 一鍵佈署或更新網站所需要使用的套件。

```ruby
npm install -g bower --save-dev
```

```ruby
# 查詢相關指令
bower help

# 查詢已經安裝的套件
bower list

# 搜尋套件
bower search <name>

# 移除已安裝的套件
bower uninstall <name>

# 升級已安裝套件
bower update <name>

# 顯示該套件的 bower.json
bower info <name>

# 透過 bower.json 來安裝相依套件
bower install
# registered package
bower install jquery
# GitHub shorthand
bower install desandro/masonry
# Git endpoint
bower install git://github.com/user/package.git
# URL
bower install http://example.com/script.js
```

* [Bower 管理網站套件的好工具](https://blog.wu-boy.com/2013/01/bower-is-a-package-manager-for-the-web/)
* [bower解決js的依賴管理](http://blog.fens.me/nodejs-bower-intro/)
* [Bower 前端套件管理工具](http://edentsai231.logdown.com/posts/198741-bower-front-end-kit-management-tool)
* [使用Bower進行前端依賴管理](http://wwsun.github.io/posts/bower-post.html)

#<span id="grunt">Grunt</span>

The JavaScript Task Runner，可以透過一些設定讓你輕鬆完成一些例行性的任務，例如壓縮檔案，編譯 coffee less，搬移到目標目錄，單元測試等等。(類似 Ruby 中的 rake)

>建構工具，主要用來運行各種任務，比如文件壓縮、合併、打包等

在新專案使用Grunt，必須要先有兩個檔案

* package.json
* Gruntfile

在package.json中，Grunt 和Grunt 任務會用的到外掛，都會列在devDependencies 中。

```ruby
npm install -g grunt-cli --save-dev
```

```ruby
# 查詢相關指令
grunt --help
```

* [Grunt](http://gruntjs.com/)
* [grunt讓Nodejs規範起來](http://blog.fens.me/nodejs-grunt-intro/)
* [Grunt 新手一日入門](http://yujiangshui.com/grunt-basic-tutorial/)
* [Grunt 系列1 基礎教學](http://andyyou.logdown.com/posts/141718-grunt)
* [Grunt 系列2 設定](http://andyyou.logdown.com/posts/142728-grunt-set-2)
* [Grunt 系列3 範例實作](http://andyyou.logdown.com/posts/143296-grunt-series-3-example-implementations)

#<span id="gulp">Gulp</span>

跟 grunt 做的事一樣，但是效能比較好

* 程式碼撰寫風格檢查、程式碼品質分析
* 最小化(Minification)、醜化(Uglify)
* 合併檔案 (Concatenation)
* 套用格式轉換 (Less, Sass, TypeScript, Babel, ... ) • 套用 Vendor prefixes
* 自動注入 JS/CSS 引用到 HTML 之中 • 更新套件版本
* 快取HTML範本
* 單元測試、整合測試、連續性整合

```ruby
npm install -g gulp --save-dev
```

主要是寫在 `gulpfile.js`

* [Gulp](http://gulpjs.com/)
* [Gulp-自動化的好幫手](http://fireqqtw.logdown.com/posts/249086-good-helper-of-gulp-automation)
* [gulp 學習筆記](https://kejyuntw.gitbooks.io/gulp-learning-notes/content/)
* [gulp 入門指南](https://github.com/nimojs/gulp-book)

#<span id="yeoman">yeoman</span>

用來自動產生網站骨架或程式碼的工具

包含以下三套工具，分別說明如下：

* yo - scaffolding tool from Yeoman 用來自動產生網站骨架或程式碼的工具
* bower - 用來管理特定網站下所使用的各式前端套件，如: jQuery
* grunt - 用來執行一些網站的自動化工作，例如單元測試、最小化、執行批次命令

```ruby
npm install -g yo --save-dev
```
[Yeoman自動建構js項目](http://blog.fens.me/nodejs-yeoman-intro/)

###以上比較

* bower - 用來做前端資源依賴管理，跟npm很像，區別在於
	* npm管理的是node模組的依賴
	* bower管理的是前端資源的依賴，如css、javascript文件等，之後就不需要手動下載和管理你的腳本文件。

* grunt/gulp - 一個幫助我們自動管理和運行JavaScript的任務之執行工具，可以用了檢查程式碼語法是否正確，壓縮程式碼，合併文件，透過Grunt可以簡化我們的工作流程。

* Yeoman - 一個 Web 應用的架構（scaffolding）工具。它提供了非常多的樣板，用來生成不同類型的 Web 應用。這些樣板稱為生成器（generator）。

參考文件：

* [從零開始nodejs系列文章](http://blog.fens.me/series-nodejs/)
* [比較grunt,npm,gulp](http://www.ifeenan.com/javascript/2014-08-05-%E6%AF%94%E8%BE%83Grunt,NPM,Gulp/)
* [透過例子對比grunt和gulp](http://jser.me/2014/03/11/%E9%80%9A%E8%BF%87%E4%BE%8B%E5%AD%90%E5%AF%B9%E6%AF%94grunt%E5%92%8Cgulp.html)
* [30 天學習 30 種新技術系列](https://segmentfault.com/a/1190000000349384)

#<span id="browserify">Browserify</span>

允許用 nodejs 的程式碼風格來定義模組，並使用在瀏覽器上，可以完全跟nodejs後端模組通用,保持單個功能模組的重用性.


* `module.exports` 來匯出模組功能
* `require` 來請求某個模組

```ruby
npm install -g browserify --save-dev
```

* [browserify](http://browserify.org/)
* [Browserify 使用指南](http://zhaoda.net/2015/10/16/browserify-guide/)
* [Node.js的browserify](http://agigi.logdown.com/note/153897-browserify-of-node-js)
* [利用browserify和gulp來建構react應用 ](http://www.ifeenan.com/javascript/2014-08-20-%E5%88%A9%E7%94%A8Browserify%E5%92%8CGulp%E6%9D%A5%E6%9E%84%E5%BB%BAReact%E5%BA%94%E7%94%A8/)

#<span id="webpack">Webpack</span>

Webpack是所謂的模組打包工具，它可以幫你把各種文件(JS、JSX、coffee、less/sass…、圖片）打包成一系列的靜態資源來使用。



* 將你的 js 檔案 Bundle 變成單一的檔案
* 在你的前端程式碼中使用 npm packages
* 撰寫 JavaScript ES6 或 ES7（需要透過 babel 來幫助）
* Minify 或優化程式碼
* 將 LESS 或 SCSS 轉換成 CSS
* 使用 HMR（Hot Module Replacement）
* 包含任何類型的檔案到你的 JavaScript

```ruby
npm install -g webpack
npm install --save-dev webpack

# 每次 build 的時候查看改變的檔案
webpack --watch

# 使用自訂的 webpack 設定檔
webpack --config myconfig.js
```

* [webpack](https://webpack.js.org/)
* [Webpack Tutorial 繁體中文 Gitbook](https://neighborhood999.github.io/webpack-tutorial-gitbook/)
* [Webpack React 入門筆記](http://blog.elaine.me/articles/React-webpack/)
* [WEBPACK入門教學筆記](http://blog.kkbruce.net/2015/10/webpack.html#.WCqPgeF97Vo)
* [Webpack - 使用 Webpack](http://skychang.github.io/2015/08/26/Webpack-First/)
* [從無到有建立 webpack 設定檔（一）](https://medium.com/html-test/%E5%BE%9E%E7%84%A1%E5%88%B0%E6%9C%89%E5%BB%BA%E7%AB%8B-webpack-%E8%A8%AD%E5%AE%9A%E6%AA%94-%E4%B8%80-42fbc76a2d37#.9kszblege)
* [從無到有建立 webpack 設定檔（二）設定樣式](https://medium.com/html-test/%E5%BE%9E%E7%84%A1%E5%88%B0%E6%9C%89%E5%BB%BA%E7%AB%8B-webpack-%E8%A8%AD%E5%AE%9A%E6%AA%94-%E4%BA%8C-%E8%A8%AD%E5%AE%9A%E6%A8%A3%E5%BC%8F-61c210d63411#.d0yrggih7)
* [從無到有建立 webpack 設定檔（三）進階設定](https://medium.com/html-test/%E5%BE%9E%E7%84%A1%E5%88%B0%E6%9C%89%E5%BB%BA%E7%AB%8B-webpack-%E8%A8%AD%E5%AE%9A%E6%AA%94-%E4%B8%89-%E9%80%B2%E9%9A%8E%E8%A8%AD%E5%AE%9A-d3d7583e61cf#.tdqkx4xlp)
* [【webpack】的基本工作流程](https://medium.com/html-test/webpack-%E7%9A%84%E5%9F%BA%E6%9C%AC%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B-585f2bc952b9#.ipr7ygu7m)
* [Webpack學習實踐系列(一)](http://blog.hugzh.com/2016/05/02/webpack%E5%AD%A6%E4%B9%A0%E5%AE%9E%E8%B7%B5%E7%B3%BB%E5%88%97-%E4%B8%80/)
* [如何使用 Webpack 模組整合工具](https://rhadow.github.io/2015/03/23/webpackIntro/)
* [深入瞭解 Webpack Plugins](https://rhadow.github.io/2015/05/30/webpack-loaders-and-plugins/)
* [webpack tutorial](https://roy-huang.com/category/webpack/)
* [Webpack 2 入門課程](https://llp0574.github.io/2016/11/29/getting-started-with-webpack2/)

#<span id="babel">Babel</span>

Babel是一個廣泛使用的轉碼器，可以將ES6程式碼轉為ES5程式碼，從而在現有環境執行。

* [Babel 入門課程](http://www.ruanyifeng.com/blog/2016/01/babel.html)
* [babel 相關名詞簡介](http://code.kpman.cc/2016/09/13/babel-%E7%9B%B8%E9%97%9C%E5%90%8D%E8%A9%9E%E7%B0%A1%E4%BB%8B/)


其他參考資料

* [終於弄懂了各種前端build工具](http://www.yidianzixun.com/article/0EmFXyTl)
* [一看就懂的前端開發環境建置入門教學](http://blog.kdchang.cc/2016/11/05/how-to-establish-modern-front-end-development-environment-tutorial/)
