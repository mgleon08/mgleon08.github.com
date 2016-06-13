---
layout: post
title: "Override Default Primary Key Id"
date: 2016-06-09 22:37:33 +0800
comments: true
categories: rails
---
有時候在處理一些 table 時，會不希望 Primary Key 是一開始預設的 id 流水號  

<!-- more -->

像是當 table 會不斷清空又重建，若是流水號，就會一直不斷的壘加上去，這時我們就會希望 table 的 Primary Key 可以自己設定  

migration 可以很方便做到這件事，也可以改成複合健(兩個 primary key，預設是無法，可以靠 composite_primary_keys  gem 來達成)

<!-- more -->

先將原本 id 移除

```ruby
def up
  remove_column :products, :id # remove existing primary key
end

def down
  add_column :products, :id, :primary_key
end
```

加上新的 primary key id

* ex1:

```ruby
create_table :products, id: false do |t|
  t.string :sku, primary: true

  t.timestamps
end
```

* ex2:

```ruby
create_table :products, id: false do |t|
  t.string :sku, primary_key: true

  t.timestamps
end
```

* ex3:

```ruby
def change
  create_table :products, id: false do |t|
    t.string :sku, null: false
    t.timestamps   
  end
  add_index :products, :sku, unique: true
  execute %Q{ ALTER TABLE "products" ADD PRIMARY KEY ("sku"); }
end
```

參考文件：  
[How to Override Default Primary Key Id in Rails](http://ruby-journal.com/how-to-override-default-primary-key-id-in-rails/)  
[composite_primary_keys](https://github.com/composite-primary-keys/composite_primary_keys)  
[Change Primary Key Issue Rails 4.0](http://stackoverflow.com/questions/23848388/change-primary-key-issue-rails-4-0)