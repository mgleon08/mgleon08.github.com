---
layout: post
title: "Association supports 方法"
date: 2016-01-31 11:54:15 +0800
comments: true
categories: rails rails語法
---

在 model 和 model 之間，經常要建立對應的關聯，rails 也提供很多 Supports 的 helper

<!-- more -->

```ruby
class User < ActiveRecord::Base
	has_many :microposts, dependent: :destroy
	has_many :active_relationships,  class_name:  "Relationship", foreign_key: "follower_id", dependent: :destroy
	has_many :passive_relationships, class_name:  "Relationship", foreign_key: "followed_id", dependent: :destroy
	has_many :following, through: :active_relationships,  source: :followed
	has_many :followers, through: :passive_relationships, source: :follower
end
```

* `through` 透過關聯來建立另一個關聯集合，用於建立多對多的關係

* `class_name` 變更關聯的類別名稱  

可以用這個方式，自己關聯自己，像是要上面，User 可以 follower 很多個 User

>rails 慣例是 model 名稱，所以不用另外加 class_name

* `foreign_key` 可以修改外鍵名稱  

>rails 外鍵慣例是關聯的 Model 名稱加上 _id 後綴

* `dependent` 

設定當物件刪除時，如何處理依賴它的資料

```ruby
class Event < ActiveRecord::Base
  has_many :attendees, :dependent => :destroy
end

#:destroy 把依賴的attendees也一併刪除，並且執行Attendee的destroy回呼
#:delete 把依賴的attendees也一併刪除，但不執行Attendee的destroy回呼
#:nullify 這是預設值，不會幫忙刪除attendees，但會把attendees的外部鍵event_id都設成NULL
#:restrict_with_exception 如果有任何依賴的attendees資料，則連event都不允許刪除。執行刪除時會丟出錯誤例外ActiveRecord::DeleteRestrictionError。
#:restrict_with_error 不允許刪除。執行刪除時會回傳false，在@event.errors中會留有錯誤訊息。
```

* `source` 

搭配through設定使用，當關聯的名稱不一致的時候，需要加上source指名是哪一種物件。

* `counter_cache ` 參考之前文章 [counter-cache](http://mgleon08.github.io/blog/2015/12/20/counter-cache/)

```ruby
class Book < ActiveRecord::Base
  has_many :pages
end

class Page < ActiveRecord::Base
  belongs_to :book, :counter_cache => true
end

#設定成 ture，就會自動去找 pages_count 欄位，若要指定欄位則是 counter_cache: :count_of_pages
```

* `inverse_of`  

關聯另一端的關聯名稱。

>belongs_to 無法與 :polymorphic 同時使用。  
>has_one 無法與 :through 或 :as 同時使用。

* `polymorphic` & `as` 參考之前文章 [polymorphic](http://mgleon08.github.io/blog/2015/12/20/ruby-on-rails-polymorphic-associations-and-sti/)

```ruby
class Comment < ActiveRecord::Base
  belongs_to :commentable, :polymorphic => true
end

class Article < ActiveRecord::Base
  has_many :comments, :as => :commentable
end

class Photo < ActiveRecord::Base
  has_many :comments, :as => :commentable
end
```
* `touch`  
touch 為 true 時，儲存或刪除關聯物件時，關聯物件的 updated_at 或 updated_on 的時間戳會自動設成當前時間

```ruby
class Order < ActiveRecord::Base
  belongs_to :customer, touch: true 
  #更改欄位 touch: :orders_updated_at
end
```

* `validate` 預設為 false，儲存物件時不會驗證關聯物件

* `primary_key ` 可以修改主鍵名稱


#Scope
* `where`

```ruby
class Order < ActiveRecord::Base
  belongs_to :customer, -> { where active: true }
end
```

* `includes`

經常性使用 `@line_item.order.customer` 就可以加上
 
```ruby
class LineItem < ActiveRecord::Base
  belongs_to :order, -> { includes :customer }
end
 
class Order < ActiveRecord::Base
  belongs_to :customer
  has_many :line_items
end
 
class Customer < ActiveRecord::Base
  has_many :orders
end
```

* `readonly`

如果設定了 readonly 選項，則關聯物件取出時為唯讀。

* `select`

select 方法可以覆寫用來取出關聯的 SELECT 子句。預設會取出所有欄位

### has_many 額外方式
條件也可透過 Hash 指定

```ruby
class Customer < ActiveRecord::Base
  has_many :confirmed_orders, -> { where confirmed: true },
                              class_name: "Order"
end
```
用 Hash 的 where，產生出來的記錄會自動使用 Hash 的作用域。  

上例中，使用 `@customer.confirmed_orders.create` 或 `@customer.confirmed_orders.build` 會建立出 confirmed 欄位為 true 的訂單

* `group` 對結果做分組

```ruby
has_many :line_items, -> { group 'orders.id' }
```

* `limit` 限制透過關聯取出物件的數量

```ruby
 has_many :recent_orders, -> { order('order_date desc').limit(100) }
```

* `offset` 指定開始從關聯取出物件的偏移量

```ruby
has_many :orders, -> { offset(11) }
```

* `order` 指定關聯物件取出後的排序方式
 
```ruby
has_many :orders, -> { order "date_confirmed DESC" }
```

* `distinct` 確保集合中沒有重複的物件
 
```ruby
has_many :articles, -> { distinct }, through: :readings
```

若想確保不插入重複的資料到資料庫（這樣取出來就確定是不重複的記錄了），應該要在資料表上新增一個唯一性的索引。

舉例來說，如果有 person_articles 資料表，想確保所有文章不重複，可加入下面這個遷移

```ruby
add_index :person_articles, :article, unique: true
```

不要使用 include? 來確保唯一性，因為多個使用者可能同時加入文章，可能會導致競態條件（Race Condition）

```ruby
person.articles << article unless person.articles.include?(article)
```

* `extending` 指定一個模組名稱，用來擴充關聯代理（association proxy）

```ruby
module FindRecentExtension
  def find_recent
    where("created_at > ?", 5.days.ago)
  end
end
 
class Customer < ActiveRecord::Base
  has_many :orders, -> { extending FindRecentExtension }
end
 
class Supplier < ActiveRecord::Base
  has_many :deliveries, -> { extending FindRecentExtension }
end
```

官方文件：  
[has_many Association Reference](http://guides.rubyonrails.org/association_basics.html#has-many-association-reference)  
[has_many Association Reference 中文](http://rails.ruby.tw/association_basics.html#has-many-%E9%97%9C%E8%81%AF%E5%8F%83%E8%80%83%E6%89%8B%E5%86%8A)  
[belongs_to 關聯手冊](http://rails.ruby.tw/association_basics.html#belongs-to-%E9%97%9C%E8%81%AF%E5%8F%83%E8%80%83%E6%89%8B%E5%86%8A)  
[has_one 關聯手冊](http://rails.ruby.tw/association_basics.html#has-one-%E9%97%9C%E8%81%AF%E5%8F%83%E8%80%83%E6%89%8B%E5%86%8A)  
[has_many 關聯](http://rails.ruby.tw/association_basics.html#has-many-%E9%97%9C%E8%81%AF%E5%8F%83%E8%80%83%E6%89%8B%E5%86%8A)

參考文件：  
[ActiveRecord - 資料表關聯](https://ihower.tw/rails4/activerecord-relationships.html)