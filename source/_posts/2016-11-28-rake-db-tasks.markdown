---
layout: post
title: "rake db tasks 指令"
date: 2016-11-28 15:04:17 +0800
comments: true
categories: rails
---

用來記錄一下 rake db 指令

<!-- more -->

```ruby
# 建立 database
rake db:create

# 刪除整個 database
rake db:drop

# 查看所有 migration 的狀態 (up 已執行過，down 尚未執行。)
rake db:migrate:status

# 還原已跑最新的 migration
rake db:rollback

# 還原已跑最新三個的 migration
rake db:rollback STEP=3

# 還原指定已跑的 migration
rake db:migrate:down VERSION=20100905201547

# 重跑目前最新的 migration
rake db:migrate:redo

# 重跑目前最新的三個 migration
rake db:migrate:redo STEP=3

# 執行 db/migrate 中還沒跑過的 migrations
rake db:migrate

# 針對設定的 migrations 版本執行
rake db:migrate VERSION=12341234

# 初始化空的資料庫
rake db:scheme:load

# 更新測試資料庫的 schema
rake db:test:prepare 

# 從目前 database 中實際的 schema 建立 db/schema.rb
rake db:schema:dump

# 從 db/schema.rb 中把 schema 建立到 databse 中
rake db:schema:load

# does db:create, db:schema:load, db:seed
rake db:setup

# does db:drop, db:create, db:schema:load
rake db:reset

# clean assets
RAILS_ENV=production bundle exec rake assets:clean

# assets precompile
RAILS_ENV=production bundle exec rake assets:precompile
```

相關 gem

* [seed_dump](https://github.com/rroblak/seed_dump)
* [yaml_db](https://github.com/yamldb/yaml_db)