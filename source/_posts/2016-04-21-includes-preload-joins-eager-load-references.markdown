---
layout: post
title: "includes preload joins eager_load references"
date: 2016-04-21 00:41:09 +0800
comments: true
categories: rails
---

rails 當中有很多方便可以做資料查詢的功能，可以好好研究一下。

<!-- more -->

#modle

```ruby
class Blog < ActiveRecord::Base
  has_many :posts

  # t.string   "name"
  # t.string   "author"
end

class Post < ActiveRecord::Base
  belongs_to :blog

  # t.string   "title"
end
```

#includes

* `includes` 主要用於可以直接將相關連的資料，在同一筆查詢，一起撈出來
* two separate queries
* 跟 preload 類似，加上 `reference` 則和 `eager_load` 類似

```ruby
Blog.includes(:posts)
Blog Load (0.1ms)  SELECT "blogs".* FROM "blogs"
Post Load (0.2ms)  SELECT "posts".* FROM "posts" WHERE "posts"."blog_id" IN (1, 2, 3)
 => #<ActiveRecord::Relation [#<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 2, name: "Blog 2", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 3, name: "Blog 3", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">]>

# 回傳所有 User 和 關聯的 posts
```
可以看到後面有 `IN (1, 2, 3)`，就是將上面一筆一筆查詢，變成這種方式一次撈出來。這樣在 `view` 中執行 `user.posts` 就不會再去資料庫查詢，因為已經都先撈出來了。

```ruby
#也可以一次 includes 多個關聯

Blog.includes(posts: :profile)
Blog.includes(posts: [:foo, :bar])
#blog 關聯到 posts，posts 關聯到 foo, bar

Blog.includes(:user, comments: [:user, { replies: [:user] }])
#更複雜的關聯
```

#preload

* 跟 includes 類似，主要差別在於無法用 where 條件去查關聯到的 table 欄位
* two separate queries

```ruby
Blog.preload(:posts)
  Blog Load (0.1ms)  SELECT "blogs".* FROM "blogs"
  Post Load (0.3ms)  SELECT "posts".* FROM "posts" WHERE "posts"."blog_id" IN (1, 2, 3)
 => #<ActiveRecord::Relation [#<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 2, name: "Blog 2", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 3, name: "Blog 3", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">]>
```

```ruby
#無法透過關聯的欄位做搜尋
Blog.preload(:posts).where(posts: { title:"Post 1-1" } )
  Blog Load (0.4ms)  SELECT "blogs".* FROM "blogs" WHERE "posts"."title" = ?  [["title", "Post 1-1"]]
=># ActiveRecord::StatementInvalid: SQLite3::SQLException: no such column: posts.title: SELECT "blogs".* FROM "blogs" WHERE "posts"."title" = ?

Blog.preload(:posts).where("posts.title = 'Post 1-1'").references(:post)
  Blog Load (0.4ms)  SELECT "blogs".* FROM "blogs" WHERE (posts.title = 'Post 1-1')
=># ActiveRecord::StatementInvalid: SQLite3::SQLException: no such column: posts.title: SELECT "blogs".* FROM "blogs" WHERE (posts.title = 'Post 1-1')

#用 includes 就沒問題，不過這就是 eager_load
Blog.includes(:posts).where(posts: { title:"Post 1-1" } )
  SQL (0.2ms)  SELECT "blogs"."id" AS t0_r0, "blogs"."name" AS t0_r1, "blogs"."author" AS t0_r2, "blogs"."created_at" AS t0_r3, "blogs"."updated_at" AS t0_r4, "posts"."id" AS t1_r0, "posts"."title" AS t1_r1, "posts"."blog_id" AS t1_r2, "posts"."user_id" AS t1_r3, "posts"."created_at" AS t1_r4, "posts"."updated_at" AS t1_r5 FROM "blogs" LEFT OUTER JOIN "posts" ON "posts"."blog_id" = "blogs"."id" WHERE "posts"."title" = ?  [["title", "Post 1-1"]]
 => #<ActiveRecord::Relation [#<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">]>

#另一種寫法 
 Blog.includes(:posts).where("posts.title = 'Post 1-1'").references(:post)
  SQL (0.3ms)  SELECT "blogs"."id" AS t0_r0, "blogs"."name" AS t0_r1, "blogs"."author" AS t0_r2, "blogs"."created_at" AS t0_r3, "blogs"."updated_at" AS t0_r4, "posts"."id" AS t1_r0, "posts"."title" AS t1_r1, "posts"."blog_id" AS t1_r2, "posts"."user_id" AS t1_r3, "posts"."created_at" AS t1_r4, "posts"."updated_at" AS t1_r5 FROM "blogs" LEFT OUTER JOIN "posts" ON "posts"."blog_id" = "blogs"."id" WHERE (posts.title = 'Post 1-1')
 => #<ActiveRecord::Relation [#<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">]>
```

#joins (inner join)

`joins` 則是關聯其他資料庫，可以進行查詢，但並不會將關聯的資料拉出來。

```ruby
Blog.joins(:posts)
  Blog Load (0.2ms)  SELECT "blogs".* FROM "blogs" INNER JOIN "posts" ON "posts"."blog_id" = "blogs"."id"
 => #<ActiveRecord::Relation [#<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 2, name: "Blog 2", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 2, name: "Blog 2", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 2, name: "Blog 2", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 2, name: "Blog 2", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 2, name: "Blog 2", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, ...]>

Blog.joins(:posts).count
   (0.1ms)  SELECT COUNT(*) FROM "blogs" INNER JOIN "posts" ON "posts"."blog_id" = "blogs"."id"
 => 15
 
#因為資料庫裡面總共有四個，但只有三個室有關聯到 posts，因此 joins 會回傳有關聯的，blog
Blog.joins(:posts).uniq.size
   (0.3ms)  SELECT DISTINCT COUNT(DISTINCT "blogs"."id") FROM "blogs" INNER JOIN "posts" ON "posts"."blog_id" = "blogs"."id"
 => 3
 
#回傳所有，有 post 的 blog
#因為同一個 blog 可能有多個 post ，這樣就會撈出重複的 blog 出來 ， 一個 post 一個 blog，因此可以用 .uniq 來去除重複的資料。
#如果是一對一就不會有這個問題了

#eager_load 則是透過 rails filter 後的
Blog.eager_load(:posts)
  SQL (0.3ms)  SELECT "blogs"."id" AS t0_r0, "blogs"."name" AS t0_r1, "blogs"."author" AS t0_r2, "blogs"."created_at" AS t0_r3, "blogs"."updated_at" AS t0_r4, "posts"."id" AS t1_r0, "posts"."title" AS t1_r1, "posts"."blog_id" AS t1_r2, "posts"."user_id" AS t1_r3, "posts"."created_at" AS t1_r4, "posts"."updated_at" AS t1_r5 FROM "blogs" LEFT OUTER JOIN "posts" ON "posts"."blog_id" = "blogs"."id"
 => #<ActiveRecord::Relation [#<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 2, name: "Blog 2", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 3, name: "Blog 3", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 4, name: "Blog 2", author: nil, created_at: "2016-04-20 16:01:54", updated_at: "2016-04-20 16:01:54">]>

#因為是 LEFT OUTER JOINed 所以沒有關聯的也會抓進來
 Blog.eager_load(:posts).count
   (0.2ms)  SELECT COUNT(DISTINCT "blogs"."id") FROM "blogs" LEFT OUTER JOIN "posts" ON "posts"."blog_id" = "blogs"."id"
 => 4
```

回傳的是所有有 `post` 的 `blog`，但並不會將 `post` 資料撈出來，只是去做比對，因此再用 `blog.posts` ，一樣會去資料庫中撈出資料。

#joins 範例
```ruby
class Category < ActiveRecord::Base
  has_many :articles
end
 
class Article < ActiveRecord::Base
  belongs_to :category
  has_many :comments
  has_many :tags
end
 
class Comment < ActiveRecord::Base
  belongs_to :article
  has_one :guest
end
 
class Guest < ActiveRecord::Base
  belongs_to :comment
end
 
class Tag < ActiveRecord::Base
  belongs_to :article
end
```

```ruby
Category.joins(:articles)
#“依文章分類來回傳分類物件”。注意到如果有 article 是相同類別，會看到重複的分類物件。若要去掉重複結果，可以使用 Category.joins(:articles).uniq。
#SELECT categories.* FROM categories
  #INNER JOIN articles ON articles.category_id = categories.id
  
Article.joins(:category, :comments)
#“依分類來回傳文章物件，且文章至少有一則評論”。有多則評論的文章將會出現很多次
#SELECT articles.* FROM articles
  #INNER JOIN categories ON articles.category_id = categories.id
  #INNER JOIN comments ON comments.article_id = articles.id
  
Article.joins(comments: :guest)
#“回傳所有有訪客評論的文章”
#SELECT articles.* FROM articles
  #INNER JOIN comments ON comments.article_id = articles.id
  #INNER JOIN guests ON guests.comment_id = comments.id

Category.joins(articles: [{comments: :guest}, :tags])
#SELECT categories.* FROM categories
  #INNER JOIN articles ON articles.category_id = categories.id
  #INNER JOIN comments ON comments.article_id = articles.id
  #INNER JOIN guests ON guests.comment_id = comments.id
  #INNER JOIN tags ON tags.article_id = articles.id
```

#joins和include的區別

* include 主要是將其他關聯的 table 一起拉進來，後續查詢時，就不會再去查
* joins 則是將兩張表合成一張（必須id有對到），再透過欄位去做塞選
* joins 為 inner join ， include 為 left outer join

```ruby
Blog.includes(:posts)
#回傳所有的 Blog，並將相關聯的 post 一併做查詢
#後續再去關聯的話就不會去 query

Blog.joins(:posts)
#查詢所有包含 user_id 的 posts ，並回傳該 post 所屬的 blog
#因此 has_many 若有很多 posts 屬於同一個 blog 就會回傳很多次重複的( 或是用 `distinct` User.joins(:posts).select('distinct users.*'))，可用 uniq 去掉，belong_to & has_one 則不會
#後續再去關聯的話，還是會去 query
```

#eager_loading
* One query, LEFT OUTER JOINed in any query rather than loaded separately.
* Works just the same as `includes` + `references` 
* 因此要小心，includers 後在接上 references 就會變成 eager_load
* Eager loading 是載入由 Model.find 回傳的物件關聯記錄的機制，將查詢數降到最低。
* Active Record 透過使用 Model.find 搭配 includes 方法，可預先指定所有會載入的關聯。有了 includes，Active Record 確保所有指定的關聯，加載的查詢減到最少。

```ruby
Blog.eager_load(:posts)
  SQL (0.3ms)  SELECT "blogs"."id" AS t0_r0, "blogs"."name" AS t0_r1, "blogs"."author" AS t0_r2, "blogs"."created_at" AS t0_r3, "blogs"."updated_at" AS t0_r4, "posts"."id" AS t1_r0, "posts"."title" AS t1_r1, "posts"."blog_id" AS t1_r2, "posts"."user_id" AS t1_r3, "posts"."created_at" AS t1_r4, "posts"."updated_at" AS t1_r5 FROM "blogs" LEFT OUTER JOIN "posts" ON "posts"."blog_id" = "blogs"."id"
 => #<ActiveRecord::Relation [#<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 2, name: "Blog 2", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 3, name: "Blog 3", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">, #<Blog id: 4, name: "Blog 2", author: nil, created_at: "2016-04-20 16:01:54", updated_at: "2016-04-20 16:01:54">]>
 
Blog.eager_load(:posts).where(name: 'Blog 1')
  SQL (0.3ms)  SELECT "blogs"."id" AS t0_r0, "blogs"."name" AS t0_r1, "blogs"."author" AS t0_r2, "blogs"."created_at" AS t0_r3, "blogs"."updated_at" AS t0_r4, "posts"."id" AS t1_r0, "posts"."title" AS t1_r1, "posts"."blog_id" AS t1_r2, "posts"."user_id" AS t1_r3, "posts"."created_at" AS t1_r4, "posts"."updated_at" AS t1_r5 FROM "blogs" LEFT OUTER JOIN "posts" ON "posts"."blog_id" = "blogs"."id" WHERE "blogs"."name" = ?  [["name", "Blog 1"]]
 => #<ActiveRecord::Relation [#<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">]>
 
Blog.eager_load(:posts).where(name: 'Blog 1').where(posts: {title: 'Post 1-1'})
  SQL (0.3ms)  SELECT "blogs"."id" AS t0_r0, "blogs"."name" AS t0_r1, "blogs"."author" AS t0_r2, "blogs"."created_at" AS t0_r3, "blogs"."updated_at" AS t0_r4, "posts"."id" AS t1_r0, "posts"."title" AS t1_r1, "posts"."blog_id" AS t1_r2, "posts"."user_id" AS t1_r3, "posts"."created_at" AS t1_r4, "posts"."updated_at" AS t1_r5 FROM "blogs" LEFT OUTER JOIN "posts" ON "posts"."blog_id" = "blogs"."id" WHERE "blogs"."name" = ? AND "posts"."title" = ?  [["name", "Blog 1"], ["title", "Post 1-1"]]
 => #<ActiveRecord::Relation [#<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">]>

# includes + references 
Blog.includes(:posts).where(name: 'Blog 1').where(posts: {title: 'Post 1-1'})
  SQL (0.2ms)  SELECT "blogs"."id" AS t0_r0, "blogs"."name" AS t0_r1, "blogs"."author" AS t0_r2, "blogs"."created_at" AS t0_r3, "blogs"."updated_at" AS t0_r4, "posts"."id" AS t1_r0, "posts"."title" AS t1_r1, "posts"."blog_id" AS t1_r2, "posts"."user_id" AS t1_r3, "posts"."created_at" AS t1_r4, "posts"."updated_at" AS t1_r5 FROM "blogs" LEFT OUTER JOIN "posts" ON "posts"."blog_id" = "blogs"."id" WHERE "blogs"."name" = ? AND "posts"."title" = ?  [["name", "Blog 1"], ["title", "Post 1-1"]]
 => #<ActiveRecord::Relation [#<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">]>
```


#references
* 只有在 includes 可以使用，主要是讓 includes 像 eager_load
* where 的這種用法只對參數是 Hash 有效。傳入參數是 SQL 片段，要使用 references 來強制連接資料表。

[#eager-loading](http://rails.ruby.tw/active_record_querying.html#eager-loading-%E9%97%9C%E8%81%AF)

```ruby
Blog.includes(:posts).where(name: 'Blog 1').references(:posts)
  SQL (0.2ms)  SELECT "blogs"."id" AS t0_r0, "blogs"."name" AS t0_r1, "blogs"."author" AS t0_r2, "blogs"."created_at" AS t0_r3, "blogs"."updated_at" AS t0_r4, "posts"."id" AS t1_r0, "posts"."title" AS t1_r1, "posts"."blog_id" AS t1_r2, "posts"."user_id" AS t1_r3, "posts"."created_at" AS t1_r4, "posts"."updated_at" AS t1_r5 FROM "blogs" LEFT OUTER JOIN "posts" ON "posts"."blog_id" = "blogs"."id" WHERE "blogs"."name" = ?  [["name", "Blog 1"]]
 => #<ActiveRecord::Relation [#<Blog id: 1, name: "Blog 1", author: "someone", created_at: "2016-04-20 14:26:01", updated_at: "2016-04-20 14:26:01">]>
```

![](http://www.codeproject.com/KB/database/Visual_SQL_Joins/Visual_SQL_JOINS_orig.jpg)

官方資料：  
[Active Record Query Interface](http://guides.rubyonrails.org/active_record_querying.html)  
[Active Record 查詢](http://rails.ruby.tw/active_record_querying.html)

參考資料：  
[網站效能](https://ihower.tw/rails4/performance.html)  
[ActiveRecord - 資料表關聯](https://ihower.tw/rails4/activerecord-relationships.html)  
[Rails使用 include 和 join 避免 N+1 query](http://motion-express.com/blog/20141028-rails-include-join-avoid-n-1-query)  
[preload, eager_load, includes, references, and joins in Rails](http://blog.ifyouseewendy.com/blog/2015/11/11/preload-eager_load-includes-references-joins/)  
[Preload, Eagerload, Includes and Joins](http://blog.bigbinary.com/2013/07/01/preload-vs-eager-load-vs-joins-vs-includes.html)  
[3 ways to do eager loading (preloading) in Rails 3 & 4](http://blog.arkency.com/2013/12/rails4-preloading/)