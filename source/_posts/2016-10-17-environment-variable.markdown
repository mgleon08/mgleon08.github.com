---
layout: post
title: "環境變數 Environment variable"
date: 2016-10-17 17:33:41 +0800
comments: true
categories: shell other
---

為什麼需要環境變數?

<!-- more -->

因為在 github 上面不會將一些敏感資料放在上面，ex: password, token..etc
所以這些資料就會在 sever 上面做設定，以免資料外漏

那要設定這些資料有兩種方式

## Config file
第一種是之前有提過的，做一個 example 設定檔，在到每個server上面個別去做設定。

* [用 Yaml 來寫文件, 設定檔](http://mgleon08.github.io/blog/2016/02/07/yaml/)

## Environment variable

在 `Unix shell` 直接設定環境變數

```ruby
#通常會用全大寫，已表示固定的變數
export TOKEN=123
```

就能在 rails 中取得

```ruby
ENV['TOKEN']
#=> 123
```

但如果是在 `Unix shell` 下直接設定的話，只會存活在該 tab 底下，開新的 tab 就會消失，因此如果希望能一直存在的話，可以設定在 `~/.zshrc` or `~/.bashrc`

>echo $SHELL 可以看是使用哪個 SHELL

## Gem

ruby 也有一些 gem 可以很方便的設定

* [figaro](https://github.com/laserlemon/figaro)
* [dotenv](https://github.com/bkeepers/dotenv)


參考文件：

* [Rails Environment Variables](http://railsapps.github.io/rails-environment-variables.html)
* [Rails的環境變數](http://braavos.me/blog/2014/08/05/rails-env/)