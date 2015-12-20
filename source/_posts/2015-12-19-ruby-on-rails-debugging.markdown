---
layout: post
title: "Ruby on Rails - Debugging"
date: 2015-12-19 11:09:06 +0800
comments: true
categories: gem rails語法
---

當發生 bug 的時候，要如何來有效率的debug!?  
今天就來介紹幾種方式，快速的找出這些 `bug!!`

<!-- more -->

#raise
直接在覺得有問題的程式碼上面，插入 `raise`  
rails 就會在該行指令處產生 [RuntimeError](http://apidock.com/ruby/Kernel/raise)，接著網頁就會進入錯誤畫面了  
下面黑色區塊就可以輸入指令，看到底哪裡出問題!!  

這是比較陽春的方式，但好處是

1. 不用特別安裝 gem
2. 也不用去 console 中 debug ， 直接在瀏覽器上就可以執行

![raise](http://i.imgur.com/W0GTWCo.png)

#[byebug](https://github.com/deivid-rodriguez/byebug)

開起新的專案時，就會內建在 development 和 test 環境的一個 gem

```ruby
group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug'
end
``` 

一樣是在覺得有問題的程式碼上面插入 `byebug`  
當跑到那個地方的時候就會停下來，接著就可以在 console 裡面輸入指令  
測試完後就輸入 `continue` 繼續跑，或是 `exit` 離開。  

但缺點是很吃效能，每次繼續跑就會變超慢，所以我都會直接整個重開XD

![byebug](http://i.imgur.com/dsaqKTB.png)


#[better_errors](https://github.com/charliesome/better_errors)

安裝方式  

```ruby
group :development, :test do
    gem 'better_errors' 
    gem 'binding_of_caller'
end
```

有點像是 raise 的進階版本，更多詳細的資訊  
只要有例外，就會顯示這種畫面

![better_errors](http://i.imgur.com/6BxsIH1.png)

1. 左上 - 檢查檔案
2. 右上 - 可以直接輸入指令來測試，像在 console 一樣
3. 右下 - 檢查數值，request的互動中是否有參數漏掉?

參考文章：  
[Debugging Rails 使用 better_errors 在瀏覽器中直接進行除錯](http://motion-express.com/blog/20141014-debugging-rails-better-errors/)

#[pry](https://github.com/pry/pry)

安裝方式

```ruby
gem 'pry'
gem 'pry-rails'
gem 'pry-nav'
```


超強的 debug 神器，一樣直接插入 `binding.pry`  
並且有顏色更加清楚，還可以像打指令 `cd @books.first` 進入某個變數裡面  
接著就可以直接打 `name` `content`，顯示變數內容

![pry](http://i.imgur.com/67g9JXM.png)

指令：  
`self` 檢查目前所在的 class 或 scope  
`next` 執行這一段 block，並在下一段 block 開始時停止  
`step` 執行這一行，並在下一行停止  
`ls、methods` 可以看目前的 class 或 scope 內有什麼樣的 variable 或 method 可以使用  
`continue` 繼續執行，如果有下一個 `binding.pry`就會停下來  
`exit` 離開 pry ，繼續執行程式

>注意!!如果是用上一篇的 pow 來執行的話，要安裝 [pry-remote](https://github.com/Mon-Ouie/pry-remote)，並且把原本的 `bindig.pry` 改成 `binding.remote_pry`

參考文章：  
[Pry ：新一代 Debug 利器](http://blog.xdite.net/posts/2012/08/13/pry-the-new-debugger)  
[Debugging Rails 沒有錯誤訊息卻還是有bug！要如何即時除錯？](http://motion-express.com/blog/20141015-debugging-rails-pry)