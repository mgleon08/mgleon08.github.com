---
layout: post
title: "Ruby on Rails - Scopes"
date: 2015-12-20 13:54:35 +0800
comments: true
categories: rails
---

在 controller 經常會用到一些資料的查詢條件
但有許多條件是同時會出現在很多地方，或是查詢條件比較複雜，無法一次就看懂在查詢什麼

這時就可以用 Scope 讓程式變得乾淨易讀，而且也可以 `串接` 來使用。

<!-- more -->

#沒帶參數

```ruby
def index
	@books = Book.order(create_at: :desc)
end
```
一般 controller 會這樣子撈出所有的資料，並且按照建立時間排序
這時就可以在 model 寫下

```ruby
class Book < ActiveRecord::Base
    scope :news_up, -> { order(created_at: :desc) }
end
```
>-> {...}是Ruby語法，等同於Proc.new{...}或lambda{...}，用來建立一個匿名方法物件

接著 controller 就只要

```ruby
def index
	@books = Book.news_up
end
```
就變得更加清楚明瞭

#帶參數

找尋建立時間小於參數的資料

```ruby
class Book < ActiveRecord::Base
  scope :recent, ->(time) {where("created_at < ?", time) }
end
```

```ruby
class Book < ActiveRecord::Base
    def self.recent(time)
        where("created_at > ? ",time)
    end
end
```
以上兩種方式皆可

接著只要在controller

```ruby
def index
	@books = Book.recent(Time.now)
end
```

#串接

```ruby
def index
	@books = Book.news_up.recent(Time.now)
end
```

```ruby
class Book < ActiveRecord::Base
  scope :recent, ->{ where(published_at: 2.weeks.ago) }
  scope :recent_red, ->{ recent.where(color: 'red') }
end
```

#default
設定所有 scope 的 default 值

```ruby
class Book < ActiveRecord::Base
    default_scope -> { order(id: :desc) }
    #or default_scope { order(id: :desc) }
end
```

官方文件：  
[Guides](http://guides.rubyonrails.org/active_record_querying.html#scopes)  
[Guides 中文](http://rails.ruby.tw/active_record_querying.html#%E4%BD%9C%E7%94%A8%E5%9F%9F)  
[apidock](http://apidock.com/rails/ActiveRecord/NamedScope/ClassMethods/scope)

參考資料：
[Scopes 作用域](https://ihower.tw/rails4/activerecord-query.html)