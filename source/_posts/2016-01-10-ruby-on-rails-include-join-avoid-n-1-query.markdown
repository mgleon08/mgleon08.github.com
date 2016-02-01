---
layout: post
title: "Ruby on Rails - 用 include 和 join 避免 N+1 query"
date: 2016-01-10 10:00:44 +0800
comments: true
categories: rails
---

在 `rails` 當中，因為 ORM (Object-relational mapping ) 的便利，可以很快速地建立起連結，但在這過程中，經常會發生 `N+1 query` 的問題，造成效能上的緩慢，因此要如何解決這個問題，是很重要的。

<!--more-->

#N+1 query

```ruby
# model
class User < ActieRecord::Base
  has_many :skills
end

class Skill < ActiveRecord::Base
  belongs_to :user
end

# controller
def index
  @users = User.all
end

# view
<% @users.each do |user| %>
 <%= user.skills %>
<% end %>
```

以上的關聯，就是透過 `User` 和 `Post 的關聯` ，在 `view` 中一筆一筆去資料庫找相關的 `post` ，而這每一筆去資料庫的動作，就會有以下的查詢，然後造成 `N+1 query` 的問題。

N+1就是指說，迴圈中查詢 N 筆資料，加上一開始的第一筆。

```ruby
Skill Load (0.2ms)  SELECT `skills`.* FROM `skills` WHERE `skills`.`user_id` = 1
Skill Load (0.2ms)  SELECT `skills`.* FROM `skills` WHERE `skills`.`user_id` = 2
Skill Load (1.6ms)  SELECT `skills`.* FROM `skills` WHERE `skills`.`user_id` = 3
```

因此要解決這個問題，就能使用以下方式。

#includes

`includes` 主要用於可以直接將相關連的資料，在同一筆查詢，一起撈出來

```ruby
User.includes(:skills)

SELECT `skills`.* FROM `skills` WHERE `skills`.`user_id` IN (1, 2, 3)

# 回傳所有 User 和 關聯的 skills
```
可以看到後面有 `IN (1, 2, 3)`，就是將上面一筆一筆查詢，變成這種方式一次撈出來。這樣在 `view` 中執行 `user.skills` 就不會再去資料庫查詢，因為已經都先撈出來了。

```ruby
也可以一次 includes 多個關聯

User.includes(skills: :profile)
User.includes(skills: [:cees, :dees])
```

#joins

`joins` 則是關聯其他資料庫，可以進行查詢，但並不會將關聯的資料拉出來。

```ruby
User.joins(:skills)

User Load (0.4ms)  SELECT `users`.* FROM `users` INNER JOIN `skills` ON `skills`.`user_id` = `users`.`id`

#回傳所有，有 skill 的 user
#因為同一個 user 可能有多個 skill ，這樣就會撈出重複的 user 出來 ， 一個 skill 一個 user，因此可以用 .uniq 來去除重複的資料。
#如果是一對一就不會有這個問題了
```
回傳的是所有有 `skill` 的 `user`，但並不會將 `skill` 資料撈出來，只是去做比對，因此再用 `user.skills` ，一樣會去資料庫中撈出資料。


官方資料：
[Active Record Query Interface](http://guides.rubyonrails.org/active_record_querying.html)
[Active Record 查詢](http://rails.ruby.tw/active_record_querying.html)

參考資料：
[網站效能](https://ihower.tw/rails4/performance.html)
[ActiveRecord - 資料表關聯](https://ihower.tw/rails4/activerecord-relationships.html)
[Rails使用 include 和 join 避免 N+1 query](http://motion-express.com/blog/20141028-rails-include-join-avoid-n-1-query)

