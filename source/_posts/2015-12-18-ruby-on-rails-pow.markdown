---
layout: post
title: "Ruby on Rails - 用 Pow 當 HTTP Server"
date: 2015-12-18 17:39:04 +0800
comments: true
categories: rails gem
---
當要啟動一個專案的時候，通常都是打 `rails s`  
就會跑在 port 3000 接著網址打 `http://localhost:3000/`，就可以打開了!

那要啟動另一個勒？ 通常有以下兩種方式
  
<!-- more -->

1. 先關掉第一個，再開啟另一個
2. 換別的 port `rails s -p 4000` 

但是萬一facebook登入，已經設定是 `http://localhost:3000/`，那就悲劇了..  
因此有另一種方式來解決這種問題 

#[pow](http://pow.cx/)  

安裝方式也相當簡單，照著官方網站

先下載到電腦上

```
curl get.pow.cx | sh
```

在指向到專案裡面

```
cd ~/.pow
$ ln -s /path/to/myapp
```

接著在 `open http://myapp.dev`  
就會啟動了!

#[powder](https://github.com/Rodreegez/powder)

powder 是一個整合好 pow 的 gem ，可以更方便快速的啟動一個專案

###安裝方式

```
gem install powder
```

###使用方式
在專案底下輸入

```
powder link
```
就會建立起連結，接著輸入

```
powder open
```
就打開了，超方便

另外一定會問到，那要怎麼看 `log`

1. `tail -f log/development.log`
2. `powder applog`

重新啟動 (通常有動到 app 以外的檔案，就會需要重啟)  

1. 	`touch tmp/restart.txt`  
2. `powder restart` 


更多指令可以到 github 上面看  
[powder](https://github.com/Rodreegez/powder)