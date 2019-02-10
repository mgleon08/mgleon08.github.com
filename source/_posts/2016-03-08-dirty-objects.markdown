---
layout: post
title: "Dirty Objects 追蹤 Model 的屬性是否有改變"
date: 2016-03-08 22:36:09 +0800
comments: true
categories: rails
---

可用來觀察，追蹤 Model 的屬性是否有改變  
可在存進資料庫前，根據是否修改，來做其他動作

<!-- more -->

```ruby
a.changed
#=> []
a.changed?
#=> false
a.title = "abc"
#=> "abc"

a.changed?
#=> true
a.changed
#=> ["title"]
a.changes
#=> {"title"=>["ha", "abc"]}

a.title_was
#=> "ha" 改變之前的值
a.title
#=> "asd"
a.title_change
#=> ["ha", "asd"]

儲存進資料庫，追蹤記錄就消失了。
a.save
#=> true
a.changed?
#=> false
a.changed
#=> []
a.changes
#=> {}
```


官方文件：  
[api - Dirty](http://api.rubyonrails.org/classes/ActiveModel/Dirty.html)  

參考文件：  
[ActiveRecord - 進階功能](https://ihower.tw/rails4/activerecord-others.html)