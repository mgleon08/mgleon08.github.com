---
layout: post
title: "Ruby on Rails - accepts_nested_attributes_for"
date: 2015-12-13 18:35:13 +0800
comments: true
categories: rails gem
---

`accepts_nested_attributes_for` 是一個蠻常會用到的語法

簡單的來說，就是可以透過這個語法，在更新 data 的時候，同時更新其他 model 裡的 data
所以並不是每個 model 都必須要有 controller 才能夠做更新的動作

<!-- more -->

這裏有兩種情境

#One-to-one

```ruby
class Book < ActiveRecord::Base
  has_one :author
  accepts_nested_attributes_for :author
end
```

```ruby
params = { book: { name: 'Harry Potter', author_attributes: { name: 'J. K. Rowling' } } }

book = Book.create(params[:book])
book.author.name # => 'J. K. Rowling'
```
透過 params 更新 Book 的 name
同時透過 `author_attributes: { name: 'J. K. Rowling' }` 更新 Author 的 name

當然 update 同樣適用

```ruby
params = { book: { author_attributes: { name: 'J. K. Rowling' } } }
book.update params[:book]
book.author.name # => 'J. K. Rowling'
```

另外值得注意的是

```ruby
accepts_nested_attributes_for :author, :allow_destroy => true, :reject_if => :all_blank
```

`:allow_destroy => true` 可以在表單增加一個 _destroy 核選塊來表示刪除
`:reject_if => :all_blank` 表示在什麼條件下就當作沒有要動作， all_blank 表示如果資料都是空的，就不執行


#One-to-many

```ruby
class Book < ActiveRecord::Base
  has_many :pages
  accepts_nested_attributes_for :page
end
```

```ruby
params = { book: {
	name: 'Harry Potter', page_attributes: [
		{ title: "Philosopher's Stone" },
		{ title: "Chamber of Secrets" }
	]
}}

book = Book.create(params[:book])
book.pages.length # => 2
book.pages.first.title # => 'Philosopher's Stone'
book.pages.last.title # => 'Chamber of Secrets'
```

#通常會搭配 fields_for 來嵌入到同一個表單

###One-to-one

```ruby
<%= form_for @book do |b| %>
  <%= b.label :name %>
  <%= b.text_field : name %>

  <%= b.fields_for :author do |a| %>
	  <%= a.label :name %>
  	  <%= a.text_field : name %>

  	  <% unless a.object.new_record? %>
     	 <%= a.label :_destroy, 'Remove:' %>
        <%= a.check_box :_destroy %>
  	  <% end %>
  <% end %>

  <%= b.submit %>
<% end %>
```

這樣就能夠透過原本是 @book 的表單，裡面再放入 author 的欄位進行更新。

###One-to-many

one-to-many 會比較麻煩，因為當新增的時候，並不知道要新增幾個，因此會動用到 jquery 的動態新增，就是可以在表單上面一直增加欄位。

不過幸好有 gem 可以解決了，以下有幾個 gem

* [nested_form](https://github.com/ryanb/nested_form) ( [網路上教學](http://motion-express.com/blog/20140722-ruby-gem-nested-form)
* [cocoon](https://github.com/nathanvda/cocoon)
* [nested_form_fields](https://github.com/ncri/nested_form_fields)

#strong parameter

最後記得要加 strong parameter
one-to-one 和 one-to-many 都要

```ruby
def book_params
    params.require(:book)
        .permit(:name, pages_attributes:[:title, :_destroy, :id])
end
```
>每個 gem strong parameter 的方式都有點不太一樣，記得要看一下

#helper

book_helper.rb

```ruby
def setup_author(book)
    book.build_author if book.author.blank?
    #one-to-one 用 build_author ， one-to-many 可以用 authors.build or authors.new
    book
end
```

如果一開始沒設定的話，在 book 表單上是看不到 author 的欄位，因為一開始還沒 build
因此要給它一個判斷，如果是 nil 就先 build_author 給它

接著用 `setup_author(@book)` 來置換 form_for 中的 @book

```ruby
<%= form_for setup_author(@book), :url => books_path do |b| %>

...

<% end %>
```

官方文件：
[Guides](http://guides.rubyonrails.org/form_helpers.html#building-complex-forms)
[Guides 中文](http://rails.ruby.tw/form_helpers.html#%E6%89%93%E9%80%A0%E8%A4%87%E9%9B%9C%E8%A1%A8%E5%96%AE)
[accepts_nested_attributes_for ](http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html)
[fields_for](http://apidock.com/rails/ActionView/Helpers/FormHelper/fields_for)