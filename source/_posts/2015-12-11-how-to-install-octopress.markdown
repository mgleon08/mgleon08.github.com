---
layout: post
title: '用 Octopress + Github Pages 來架設 blog'
date: 2015-12-11 23:15:37 +0800
comments: true
categories: blog other
---

最近想說要開始來寫 blog，於是在網路上找了幾個之後，最後決定使用 Octopress。

1. 使用 Markdown 語法，比起網路上的編輯器更加方便 (對寫 coding 而言拉。
2. 直接放在 Github 上，完全不需費用。
3. 可以用 git 來做版本控制，即使網路上的消失，本機還是可以做保留。
4. 一開始雖然比一般 blog 更加挑戰性(因為全部都要自己安裝 XD)，不過也藉此訓練自己。

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

> 像筆者帳號是 mgleon08 就輸入 mgleon08.github.com
> 之後網址就是 http://mgleon08.github.io/

#佈署到 Github 上

```
rake setup_github_pages  ＃ 之後輸入 git@github.com:mgleon08/mgleon08.github.com.git
rake generate
rake deploy
```

`rake setup_github_pages` 將 branch 從 master 切換到 source
[官方說明](http://octopress.org/docs/deploying/github/)
`rake generate` 產生靜態 html 檔案
`rake deploy` 部署到 github

#Push 到 Github

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

clone 下來後

```
rake install['主題名稱']  # for zsh, use: rake install\['主題名稱'\]
rake generate
```

#基本網站設定
設定主要都在 `_config.yml`，可以更改網站標題，作者名稱，還可以加入 facebook 按讚，DISQUS 回覆等功能
修改後要記得 `rake generate` 才會有效

#新的頁面

```
rake new_page["title"]
```

再去 `source/_includes/custom/navigation.html` 新增連結即可。

#撰寫文章

```
rake new_post["title"]
```

Octopress 會幫你產生新的檔案 `2015-12-11-title.markdown` 到 source/\_posts/下面，有發現後面是用 markdown 嗎?

沒錯，Octopress 主要是用 Markdown 語法來撰寫，筆者是使用 [MacDown](http://macdown.uranusjr.com/) 這套來做編輯（蠻多人是推薦另外一個 [mou](http://25.io/mou/)

> 對 Markdown 語法還不了解的可以到 [Markdown.tw](http://markdown.tw/) 看看

#發表文章

- 撰寫好後先 `rake preview` 預覽一下畫面
- 確認先 `rake generate`
- 再 `rake deploy` 這樣網路上的就會更新了
- 記得最後還是要 `git push` 到 Github
- (目前使用 ruby 2.4.6)

這樣就可以開始寫文章囉~

- [octopress 官方網站](http://octopress.org/docs/blogging/)
- [Octopress 課程目錄](http://shengmingzhiqing.com/blog/octopress-tutorials-toc.html/)
