---
layout: post
title: "用 ruby 做網頁爬蟲"
date: 2016-02-07 15:21:59 +0800
comments: true
categories: ruby
---

網頁爬蟲是一個蠻常聽到的名詞，簡單的來說就是可以透過程式，去分析網站頁面，將想要的資訊抓下來!

<!-- more -->

###wget
command line 下載檔案的指令  
mac 本身沒有內建要另外安裝。

```
brew install wget
```

```ruby
require "open-uri" #open-uri 把一個網頁當一個檔案來打開 stb-lib
require "nokogiri" #解析 html gem

html = open("http://ezprice.com.tw/").read
doc = Nokogiri::HTML(html)
ans = []

doc.search('img').each do |i|
  ans << i.attr('src')
end

#將相對路徑改成絕對路徑
temp_ans = ans.map do 
  |url| url.match(/^http/) ? url : "http://ezprice.com.tw/#{url}"
end

#透過 wget 下載檔案到目前的資料夾
temp_ans.each do |full_url|
  `wget #{full_url}`
end
```


`open-uri` 只能一個網址  
`curb` 比較豐富


官方文件：  
[open-uri](http://ruby-doc.org/stdlib-2.3.0/libdoc/open-uri/rdoc/index.html)  
[nokogiri](http://www.nokogiri.org/)

參考文件：  
[RailsFun](https://www.youtube.com/watch?v=6XUnYRB0Zpo&list=PLJ6M-k9dQEQ3VsyOZQwjZ5GdjaLJH3eB_)  
[Ruby Crawler 輕輕鬆鬆做個 Ruby 爬蟲機器人](http://tonytonyjan.net/slides/2014-07-03-simple-crawler/)  
[第十一篇 - 第一次自幹爬蟲就上手 - 使用 Ruby](http://yukaihuang93.logdown.com/posts/243459/how-to-write-web-crawler-for-the-first-time-using-ruby)