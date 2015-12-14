---
layout: post
title: "用 Octopress + Github Pages 來架設 blog"
date: 2015-12-11 23:15:37 +0800
comments: true
categories: Octopress blog 教學
---

最近想說要開始來寫blog，於是在網路上找了幾個之後，最後決定使用Octopress。 

1. 使用 Markdown 語法，比起網路上的編輯器更加方便 (對寫coding而言拉。   
2. 直接放在 Github 上，完全不需費用。
3. 可以用git來做版本控制，即使網路上的消失，本機還是可以做保留。
4. 一開始雖然比一般blog更加挑戰性(因為全部都要自己安裝XD)，不過也藉此訓練自己。

<!--More-->

#Install Octopress
先到 [Octopress](https://github.com/imathis/octopress) 將檔案 `git clone`下來  


```
git clone git://github.com/imathis/octopress.git octopress
cd octopress
bundle install
```

#開新的 Github Repository

到 [Github](https://github.com/) 點選右上角 New repository, Repository name 輸入自己帳號的名稱  
>像筆者帳號是 mgleon08 就輸入 mgleon08.github.com  
>之後網址就是 http://mgleon08.github.io/

#佈署到Github上
```
rake setup_github_pages  ＃ 之後輸入 git@github.com:mgleon08/mgleon08.github.com.git
rake generate
rake deploy
```
`rake setup_github_pages` 將branch從master切換到source
[官方說明](http://octopress.org/docs/deploying/github/)  
`rake generate` 產生靜態html檔案  
`rake deploy` 部署到github

#Push到Github
```
git add .
git commit -m "first commit"
git push origin source
```


#在本機端預覽
```
rake preview
```
	
接著打開 `http://localhost:4000` 就可以看到了!!

#更換主題
本站使用的主題 [whitespace](https://github.com/lucaslew/whitespace)  
其他主題 [other](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes)  

clone下來後
```
rake install['主題名稱']  # for zsh, use: rake install\['主題名稱'\]
rake generate
```

#基本網站設定
設定主要都在 `_config.yml`，可以更改網站標題，作者名稱，還可以加入facebook按讚，DISQUS回覆等功能  
修改後要記得 `rake generate` 才會有效

#撰寫文章
```
rake new_post["title"]
```

Octopress會幫你產生新的檔案 `2015-12-11-title.markdown` 到source/_posts/下面，有發現後面是用markdown嗎?  

沒錯，Octopress 主要是用 Markdown 語法來撰寫，筆者是使用 [MacDown](http://macdown.uranusjr.com/) 這套來做編輯（蠻多人是推薦另外一個 [mou](http://25.io/mou/) 

>對Markdown語法還不了解的可以到 [Markdown.tw](http://markdown.tw/) 看看

#發表文章

*	撰寫好後先 `rake preview` 預覽一下畫面  
*	確認先 `rake generate`
*	再 `rake deploy` 這樣網路上的就會更新了  
*	記得最後還是要 `git push` 到 Github

這樣就可以開始寫文章囉~

[octopress官方網站](http://octopress.org/docs/blogging/)


