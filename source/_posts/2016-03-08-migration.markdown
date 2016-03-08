---
layout: post
title: "Migration 中新增欄位 + 指令"
date: 2016-03-08 22:50:55 +0800
comments: true
categories: rails
---

當已經上線的網站，需要新的欄位，而欄位又需要有值，這時就可以直接在 `migration` 下指令去跑。

<!-- more -->

在已經啟動的 project 新增欄位時，可以額外下指令，去給初始值。

```ruby
rails g migration add_Date_Time_To_User
```

#execute

```ruby
class AddDateTimeToUser < ActiveRecord::Migration
  def up
    add_column :users, :confirmed_at, :datetime

    execute("UPDATE users SET confirmed_at = NOW()")
  end

  def down
    remove_columns :users, :confirmed_at
  end
end
```

在 `migration` 重算 `counter cache`  

[counter cache](http://mgleon08.github.io/blog/2015/12/20/counter-cache/)

```ruby
class AddPostsCountToTopic < ActiveRecord::Migration
  def change  
    add_column :topics, :posts_count, :integer, :default => 0
    
    Topic.pluck(:id).each do |i|
      Topic.reset_counters(i, :posts) # 全部重算一次
    end
  end
end
```
參考文件：  
[Active Record - 資料庫遷移(Migration)](https://ihower.tw/rails4/migrations.html)