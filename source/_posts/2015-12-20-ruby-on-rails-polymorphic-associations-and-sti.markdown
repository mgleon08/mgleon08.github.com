---
layout: post
title: "Ruby on Rails - Polymorphic Associations and STI"
date: 2015-12-20 14:47:50 +0800
comments: true
categories: rails
---

Polymorphic Associations 和 STI，一開始實在不太懂這兩個的差別是什麼？
感覺功能都差不多，但仔細研究後發現，其實兩個是完全不同的東西

<!-- more -->

#STI 單一表格繼承(Single-table inheritance)

簡單的來說，就是 `子類別` 繼承 `父類別` 的表格欄位和方法
在 Rails 慣例中，只要在父類別加上 `type` 欄位，就可以使用了

通常在資料一樣，行為不一樣的時候可以使用

以下就是三個 model 共用 User 的表格，`Guest` 和 `Member` 不需要在建立自己的資料表

```ruby
class User < ActiveRecord::Base
	validates_presence_of :name
end

class Guest < User
end

class Member < User
end
```

接著到 console 實際看看

```ruby
guest = Guest.create( :name => "leon")
guest.type # "Guest"
guest.id # 1
member = Member.create( :name => "Rails Team" )
member.type # "Member"
member.id # 2
```

![STI](http://i.imgur.com/h3vf6bE.png)
會發現自動就會在 type 記錄是哪個 model，並且 id 會繼續延續下來（因為是存在同一張資料表上）

>建議繼承的欄位是一致的在使用這個功能，因為交集的欄位不多的話，就會使得很多空間被浪費掉

要關閉STI，在父類別加上self.abstract_class = true

```ruby
class User < ActiveRecord::Base
	validates_presence_of :name
	self.abstract_class = true
end

class Guest < User
end

class Member < User
end
```
這樣 Guest 和 Member 就必須有自己的資料表了。


官方文件：  
[Guides 中文](http://rails.ruby.tw/association_basics.html#%E5%96%AE%E8%A1%A8%E7%B9%BC%E6%89%BF)

參考文件：  
[Ruby on Rails 實戰聖經](https://ihower.tw/rails4/activerecord-others.html)  
[Single Table Inheritance with Rails 4 (Part 1)](https://samurails.com/tutorial/single-table-inheritance-with-rails-4-part-1/)

#多型關聯(Polymorphic Associations)

看字面上的意思，就知道是用在關聯上的，例如 `留言` ，Article 上面可以留言，Photo 上面可以留言，通常都會直接分別建立 `ArticleComment`，`PhotoComment`的 Model。

但如果用 Polymorphic Associations 只要在另外建立一個 `Comment model` ，並加上 integer 的 _id 外部鍵和 string 的 _type 欄位說明是哪一種 Model，即可將所有留言都儲存在這裡。

```ruby
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.text :content
      t.integer :commentable_id
      t.string :commentable_type
      #可改成 t.belongs_to :commentable, polymorphic: true, index: true 取代
      #或是 t.references :commentable, polymorphic: true, index: true 取代

      t.timestamps
    end
  end
end
```

model 中的設定

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
之後再 console

```ruby
a = Article.create #id=>1
a.comments.create(:content => "First Comment")
a.comments.create(:content => "First Comment")

b = Photo.create #id=>1
b.comments.create(:content => "First Comment")
b = Photo.create #id=>2
b.comments.create(:content => "First Comment")

# 也可以透過 commentable 反向回查關連的物件
comment.commentable => #<Article id: 1, ....>
```

![Polymorphic Associations](http://i.imgur.com/9t6JGzp.png)

###eager-load-polymorphic

```ruby
#單純去 include 是沒問題的
Comment.includes(commentable: :user)

#當要做排序就會出現
#ActiveRecord::StatementInvalid: Mysql2::Error: Unknown column 'users. commentable_type' in 'where clause':
Comment.includes(commentable: :user).order("users.id DESC")
```

因為用了 Polymorphic，所以在用像是 includes 關聯的時候，因為不知道 _id 會指到哪個 table(並不存在 commentable)，所以會出現錯誤 
 
`Can not eagerly load the polymorphic association : commentable`  

這是就要另外加上以下這兩個關聯才行，明確讓他知道關聯到哪個 table

```ruby
class Comment < ActiveRecord::Base
  belongs_to :commentable, :polymorphic => true
  belongs_to :article, -> { where(comments: {contract_type: 'Article'}) }, foreign_key: 'commentable_id'
  belongs_to :photo, -> { where(comments: {contract_type: 'Photo'}) }, foreign_key: 'commentable_id'
end
```

```ruby
#就能夠透過 include 明確知道是要關聯到哪個 table，再根據該 table 去做排序
Comment.includes(article: :user).order("articles.id DESC")
```

[eager-load-polymorphic](http://stackoverflow.com/questions/16123492/eager-load-polymorphic)

官方文件：
  
* [Guides](http://guides.rubyonrails.org/association_basics.html#polymorphic-associations)  
* [Guides 中文](http://rails.ruby.tw/association_basics.html#%E5%A4%9A%E5%9E%8B%E9%97%9C%E8%81%AF)

參考資料：
  
* [Ruby on Rails 實戰聖經](https://ihower.tw/rails4/activerecord-relationships.html)  
* [eager-load-polymorphic](http://stackoverflow.com/questions/16123492/eager-load-polymorphic)
* [Single-table inheritance vs. polymorphic associations in Rails: find what works for you](https://medium.freecodecamp.org/single-table-inheritance-vs-polymorphic-associations-in-rails-af3a07a204f2)
