---
layout: post
title: "Ruby on Rails staging 環境和部署"
date: 2016-01-13 22:59:11 +0800
comments: true
categories: deploy
---

部署上 server 後，通常會需要在一個 staging server
在上線之前，先部署到 staging server ，以防一些本機測試不到的一些東西。

<!-- more -->

###新增environments/staging.rb
新增 `config/environments/staging.rb` 環境設定
內容同 `config/environments/production.rb`

###新增deploy/staging.rb
新增   `config/deploy/staging.rb`
內容同 `config/deploy/production.rb`

佈署 ip 看看是不是同一台伺服器

###server新增設定檔
server 上新增 `/opt/nginx/conf/vhost/staging.conf` 設定檔，內容跟 production 用的 nginx 設定檔一樣
除了需要加一行 `rack_env staging;`  因為預設是 `rack_env prodcution;`

###設定網址和部署目錄
如果 staging 和 production 同一台 server 的話
網址(domain name)和佈署目錄得要不一樣：
上述的 nginx vhost 設定，裡面

1. server_name 的網址要改
2. root 目錄位置要改

###搬移指令
將專案本來 `config/deploy.rb` 裡面的
`set :deploy_to, '/home/deploy/shopping'` 設定搬到
`config/deploy/staging.rb` 和 `config/deploy/production.rb`
並且改成不同目錄

```ruby
set :deploy_to, '/home/username/staging'
#指定目錄
set :branch, 'staging'
#指定branch
```

###Push code to github
`git push`

###檢查部署缺少什麼檔案
`cap staging deploy:check`
到 server 上新增 shared/config/database.yml 和 secrets.yml，注意 yml 第一層要改成 staging。

如果有其他 yml 設定也需要一併設定
(例如 facebook.yml 和 email.yml 裡面要多加 staging)

###新增staging資料庫
在 server 上新增 staging 用的資料庫
`mysql -u root -p`
`CREATE DATABASE ["databasename"] CHARACTER SET utf8;`

###部署
`cap staging deploy`

###重開nginx
`sudo service nginx restart`

參考文件：

* [【Ruby on Rails 】 設定自訂環境(Environment)](http://fireqqtw.logdown.com/posts/205866-ruby-on-rails-set-custom-environment)
* [Rails 佈署](http://pm.5fpro.com/projects/public-wiki/wiki/Rails_%E4%BD%88%E7%BD%B2)
* [網站佈署](https://ihower.tw/rails4/deployment.html)
* [3 招實用 Asset Pipeline 加速術](http://blog.xdite.net/posts/2012/07/09/3-way-to-speedup-asset-pipeline)
* [Capistrano 3 实现 Rails 自动化部署](https://ruby-china.org/topics/18616)