---
layout: post
title: "Transactions 交易"
date: 2016-01-31 10:58:13 +0800
comments: true
categories: rails
---

當兩件事情必須確實執行完畢，才存取到資料庫！

<!-- more -->

#Transactions交易

像是銀行匯款，必須一方確實扣款，另一方確實有新增款項，這筆交易在算成功!

```ruby
ActiveRecord::Base.transaction do
  User.create!(:name => 'hello')
  Feed.create!
end
```

在 transaction 裡面必須使用加上 `!` 才會丟例外，讓交易失敗。

另外，資料要在 transaction 完成後，才會存取到資料庫，因此有用 `after_save` 回呼，可能就會失敗。

因此必須改用 `after_commit`這個回呼，才能確保讀取到交易完成後的資料。

```ruby
class User < ActiveRecord::Base  def foo    self.class.transaction do￼￼      self.update!(name: 'bar')    end 
  endend
```

參考文件：  
[ActiveRecord - 進階功能](https://ihower.tw/rails4/activerecord-others.html)
