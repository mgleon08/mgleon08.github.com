---
layout: post
title: "Capistrano 自動化部署設定"
date: 2017-03-30 12:51:42 +0800
comments: true
categories: deploy
---
用 Capistrano 做自動化部署

<!-- more -->

記得先把 server 相關的設定都設定好。
還有 SSH 相關的可以參考

* [遠端 SSH 免密碼登入(key)設定](http://mgleon08.github.io/blog/2015/12/14/ssh-no-password/)

### Gemfile

```ruby
group :development do
  gem 'capistrano'
  gem 'capistrano-rails'
  gem 'capistrano-bundler'
  gem 'capistrano-rbenv' # 如果是用 rbenv 做 ruby 版控才需要
end
```

### Generate Capistrano Config Files

```ruby
# 若是要不同的名稱可以加上 STAGES=sandbox,staging
# 預設為 production & staging
cap install

# Capfile
# config/deploy.rb
# config/deploy/<stage_name>.rb
```


### deploy.rb 設定檔

可以將共用的設定放在 `config/deploy.rb` 其他個別環境的就放在 `config/deploy/production.rb` `config/deploy/staging.rb` 
等等

```ruby
# config/deploy.rb

# lock Capistrano Version
lock '3.6.1'

# application name
set :application, 'my_app_name'

# which repo to pull from when deploying
set :repo_url, 'git@example.com:me/my_repo.git'

# ask which brnach to pull from when deploying
# Default branch is :master
ask :branch, `git rev-parse --abbrev-ref HEAD`.chomp

# location to put your app on server
# Default deploy_to directory is /var/www/my_app_name
set :deploy_to, '/home/deploy/my_app_name'

# rbenv settings, version and path
set :rbenv_ruby, '2.2.3'

# these files will be symlinked to APP_PATH/shared/
# Default value for :linked_files is []
set :linked_files, %w(config/database.yml config/secrets.yml)

# these directories will be symlinked to APP_PATH/shared/
# Default value for linked_dirs is []
set :linked_dirs, %w(log tmp/pids tmp/cache tmp/sockets public/system)

# setting $PATH
# Default value for default_env is {}
set :default_env, { path: "/opt/ruby/bin:$PATH" }

# how many releases you want to keep on server
set :keep_releases, 3

# 也可以在這裡設定一些 hook
# 範例
namespace :deploy do
 desc "Run custom task"
  after :published, :task do # 在什麼 flow 之後
    on release_roles :all do # 可以執行的主機權限
      within "#{current_path}" do # 在 current 目錄底下
        with rails_env: :staging do # 在 staging 環境
          execute :rake, "task_run"
        end
      end
    end
  end
end
```

```ruby
# config/deploy/production.rb
# 用來設定連哪台 server, user 是誰, 該主機有什麼權限
# db: 處理 db 相關，像是跑 migrate, web: 跑 asset:precompile
server 'example.com', user: 'deploy', roles: %w{app db web}, my_property: :my_value
```

### Capfile 設定檔

require 需要的檔案

```ruby
# Capfile

# Load DSL and set up stages
require "capistrano/setup"

# Include default deployment tasks
require "capistrano/deploy"

# Include tasks from other gems included in your Gemfile
#
# For documentation on these, see for example:
#
#   https://github.com/capistrano/rvm
#   https://github.com/capistrano/rbenv
#   https://github.com/capistrano/chruby
#   https://github.com/capistrano/bundler
#   https://github.com/capistrano/rails
#   https://github.com/capistrano/passenger
#
# require 'capistrano/rvm'
# require 'capistrano/rbenv'
# require 'capistrano/chruby'
# require 'capistrano/bundler'
# require 'capistrano/rails/assets'
# require 'capistrano/rails/migrations'
# require 'capistrano/passenger'

# Load custom tasks from `lib/capistrano/tasks` if you have any defined
Dir.glob("lib/capistrano/tasks/*.rake").each { |r| import r }
```

# 新增environments/staging.rb

新增 `config/environments/staging.rb` 環境設定 內容同 `config/environments/production.rb`

# 產生 releases & shared 目錄
本機執行 `cap production deploy:check`，就會自動登入遠端的伺服器，在登入的帳號下新建releases和shared這兩個目錄

* releases是每次佈署的檔案目錄
* shared目錄則是不同佈署目錄之間會共用的檔案。

參考網站：

* [capistrano](https://github.com/capistrano/capistrano)
* [capistrano - Before / After Hooks](http://capistranorb.com/documentation/getting-started/before-after/)
* [capistrano - Flow](http://capistranorb.com/documentation/getting-started/flow/)
* [Capistranorb - configuration](http://capistranorb.com/documentation/getting-started/configuration/)
* [Capistrano - Variables](http://www.freelancingdigest.com/articles/capistrano-variables/)
* [網站佈署](https://ihower.tw/rails/deployment.html)
