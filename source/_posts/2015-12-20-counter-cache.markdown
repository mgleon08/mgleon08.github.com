---
layout: post
title: "Ruby on Rails - Counter_cache"
date: 2015-12-20 12:16:23 +0800
comments: true
categories: rails rails語法
---

通常要計算 has_many 關聯資料的數量時，就會用以下方式。

```ruby
@book.pages.size
```
<!-- more -->

但這樣就會產生很多SQL count查詢，造成效能上的問題

```ruby
SELECT * FROM `pages` LIMIT 5 OFFSET 0
SELECT count(*) AS count_all FROM `pages` WHERE (`pages`.book_id = 1 )
SELECT count(*) AS count_all FROM `pages` WHERE (`pages`.book_id = 2 )
SELECT count(*) AS count_all FROM `pages` WHERE (`pages`.book_id = 3 )
```

因此可以在 Book Model 上新增一個欄位，`pages_count`，用來記錄該 book 的 page 數量，之後撈資料，就直接撈這個數字就可以了。

```ruby
def change
	add_column :books, :pages_count, :integer, default: 0
end

#型態是 integer，記得設定 default 值
```

接著編輯兩邊的 model，Rails 內建有 counter_cache helper，可以直接去做計算!

```ruby
class Book < ActiveRecord::Base
  has_many :pages
end

class Page < ActiveRecord::Base
  belongs_to :book, :counter_cache => true
end

#設定成 ture，就會自動去找 pages_count 欄位，若要指定欄位則是 counter_cache: :count_of_pages
```

這樣之後下 `@book.pages.size` 的指令時 Rails 就會預設去找 `pages_count` 的欄位，不用重新下 SQL指令計算。

也可以直接下 `@book.pages_count`

最後注意，必須要用 `create` 和 `destroy` 才會 call back 更改數字，`delete` 則不會。

官方文件：  
[Guides](http://guides.rubyonrails.org/association_basics.html#detailed-association-reference)