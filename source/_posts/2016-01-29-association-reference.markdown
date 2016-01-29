---
layout: post
title: "Association Reference 建立關聯"
date: 2016-01-29 21:33:02 +0800
comments: true
categories: rails 
---

在 model 和 model 之間，經常要建立對應的關聯，而 rails 又有相當多的關聯方式，還有 method

<!-- more -->

#has_many 建立關聯

`User` has_many `Post`  
可以用下面方法，將兩個 model 關聯起來。

```ruby
class User < ActiveRecord::Base
  has_many :posts
end
```

```ruby
@user.posts 
#回傳所有關聯物件的陣列，沒有則回傳空陣列

@user.posts << Post.find(1) 
#透過將外鍵設為加入物件的主鍵

@user.posts.delete(object, ...) 
#透過將外鍵設為 NULL，從關聯集合中移除一個或多個物件，會回呼 dependent 設定

@user.posts.destroy(object, ...) 
#呼叫 destroy 來移除物件，不管 dependent 設定

@user.posts = (objects) 
#根據提供的物件來決定要刪除還是新增

@user.post_ids 
#回傳所有關聯的 id 陣列

@user.post_ids=([1,2,3])
#更改集合擁有物件的 ID，根據所提供的主鍵值來決定要刪除還是新增

@user.posts.clear
#移除集合中的所有物件。若有設定 dependent: :destroy 選項，則會 destory 關聯物件；若設定的是 dependent: :delete_all 選項，則會直接從資料庫刪除關聯物件；其他情況會將外鍵設為 NULL

@user.posts.empty?
#在集合沒有任何關聯物件時回傳 true

@user.posts.size
#回傳集合中物件的數量

@user.posts.find(...)
#在集合中查詢物件。語法和選項與 ActiveRecord::Base.find 相同

@user.posts.where(...)

@open_orders = @customer.orders.where(open: true) # No query yet
@open_order = @open_orders.first # Now the database will be queried
#根據提供的條件來查找物件，預設是惰性載入，僅在需要用到物件才會去資料庫做查詢。

@user.posts.exists?(...)
#依提供的條件檢查物件存在集合裡。語法和選項與 ActiveRecord::Base.exists? 相同

@user.posts.build(attributes = {}, ...)
#回傳一個或多個新關聯物件。這些物件由傳入的屬性來初始化，同時會自動設定外鍵。但關聯物件仍未儲存至資料庫

@user.posts.create(attributes = {})
#回傳關聯類型的新物件。 這個物件透過傳入的屬性來初始化，同時會自動設定外鍵。一旦通過所有 Model 的驗證規則時，便把此關聯物件存入資料庫

@user.posts.create!(attributes = {})
#驗證失敗時會拋出 ActiveRecord::RecordInvalid 異常
```

官方文件：  
[has_many Association Reference](http://guides.rubyonrails.org/association_basics.html#has-many-association-reference)  
[has_many Association Reference 中文](http://rails.ruby.tw/association_basics.html#has-many-%E9%97%9C%E8%81%AF%E5%8F%83%E8%80%83%E6%89%8B%E5%86%8A)