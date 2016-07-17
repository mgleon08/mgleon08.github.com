---
layout: post
title: "Retrieving Multiple Objects in Batches 批次取出多筆記錄"
date: 2016-06-22 00:32:45 +0800
comments: true
categories: rails
---

#Retrieving Multiple Objects in Batches

當要處理多筆資料時，一般都會用 User.all 去取資料  

<!-- more -->

但是當資料相當龐大，一次取出那麼多會非常佔用記憶體  
因此有以下解決方式，一次拿一部分（預設為1000）

#find_each

* 一筆一筆放入區塊。

```ruby
User.find_each(start: 2000, batch_size: 5000) do |user|
  NewsMailer.weekly(user).deliver_now
end

#:start 選項可以設定批次的起始 ID。
#:batch_size 選項允許你在將各筆記錄傳進區塊前，指定一批要取多少筆記錄。
```

#find_in_batches

* 取出記錄放入陣列傳至區塊

```ruby
Invoice.find_in_batches do |invoices|
  export.add_invoices(invoices)
end

#和 find_each 一樣的選項 :batch_size 與 :start。
```

官方資料：  
[Active Record Query Interface](http://guides.rubyonrails.org/active_record_querying.html)  
[Active Record 查詢](http://rails.ruby.tw/active_record_querying.html)