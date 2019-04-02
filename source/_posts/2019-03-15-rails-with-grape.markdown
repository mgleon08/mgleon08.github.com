---
layout: post
title: "Rails with grape"
date: 2019-03-15 23:04:35 +0800
comments: true
categories: rails
---

<!-- more -->

Grape 是一個用來建立 API 的 framework，不過在 rails5 之後就有支援 `API mode`，不過還是來了解一下，實際上來用應該還是用 rails 內建的 `API mode`

> An opinionated framework for creating REST-like APIs in Ruby.

好處

* 採用獨特的 DSL 來簡化建立API的流程
* 支援 swagger 文件自動生成

# Create Project

```ruby
rails new demo_rails_grape -T -d mysql
```

```ruby
rails g model User name:string email:string tel:string
```

### rails c

```ruby
User.create(name: "user1", email: "text1@gmail.com", tel: 111)
User.create(name: "user2", email: "text2@gmail.com", tel: 222)
```

or

### seed

```ruby
rake db:seed
```

# Grape

grape 設定

```ruby
# Gemfile
gem "grape", "~> 1.1.0"
```

```ruby
# routes
Rails.application.routes.draw do
	mount APIBase => "/api"
end
```

```ruby
# /api/api_base.rb
class APIBase < Grape::API
  # general settings
  format :json
  content_type :json, "application/json"
  default_error_status 400

  mount V1::Base
end
```

```ruby
# /api/v1/base.rb
module V1
  class Base < APIBase
    version "v1", using: :path
    mount User
  end
end
```

```ruby
# /api/v1/base/user
module V1
  class User < Base
    get :user do
      present :users, "leon"
    end
  end
end
```

```ruby
# rs -p 4321
http://localhost:4321/api/v1/user
```

# Grape Swagger

```ruby
# Gemfile
gem 'grape-swagger'
gem 'grape-swagger-rails'
```

```ruby
# routes
if Rails.env.development?
  mount GrapeSwaggerRails::Engine => '/apidoc'
end
```

```ruby
# config/initializers/grape_swagger.rb
if Rails.env.development?
  GrapeSwaggerRails.options.app_url = '/api/v1/'
  GrapeSwaggerRails.options.url = 'swagger_doc.json'
  GrapeSwaggerRails.options.app_name = 'RailsGrape Swagger'
  # Set docExpansion with "none" or "list" or "full", default is "none".
  GrapeSwaggerRails.options.doc_expansion = "list"
  GrapeSwaggerRails.options.supported_submit_methods = ['get']
end
```

```ruby
# api/v1/base/user
module V1
  class User < Base
    get :user do
      users = ::User.all
      present :users, users
    end
  end
end
```

```ruby
# /api/v1/base.rb
module V1
  class Base < APIBase
    version 'v1', using: :path

    mount User

    if Rails.env.development?
      add_swagger_documentation(
        api_version: 'v1',
        mount_path: 'swagger_doc',
        hide_format: true,
        hide_documentation_path: true,
        info: {
          title: 'API Doc',
          description: '基本使用規則'
        }
      )
    end
  end
end

```

```ruby
# rails s -p 4321
http://localhost:4321/apidoc
# json
http://localhost:4321/api/v1/swagger_doc.json
```

# grape-entity

JSON 的 field 內容就會是對應 entity 中配置的欄位，能方便的配置需要返回什麼欄位

> An API focused facade that sits on top of an object model.

新增 Gem

```ruby
# Gemfile
gem "grape-entity", "~> 0.7.1"
gem "grape-swagger-entity", "~> 0.3.0"
```

新增 entities

```ruby
# /api/v1/entities/user_result.rb
module V1::Entities
  class UserResult < Grape::Entity
    expose :name, documentation: { required: true, type: 'String', desc: '錯誤訊息' }
  end
end
```

在 swagger 新增資訊，加上 entity

```ruby
# /api/v1/user.rb
module V1
  class User < Base
    # users
    desc "`回傳所有 User 資訊`" do
      failure [
        [400, "`找不到任何 user`"],
        [500, "`unknown`"],
      ]
    end

    get :users do
      users = ::User.all
      present :users, users, with: V1::Entities::UserResult
    end

    # user
    desc "`帶 name 回傳該 User 資訊`" do
      success model: V1::Entities::UserResult, examples: {
        'application/json': {
          name: 'user1'
        },
      }
      failure [
        [400, "`找不到任何 user`"],
        [500, "`unknown`"],
      ]
    end

    params do
      optional :name, type: String, allow_blank: false, documentation: {
        description: "name",
        example: "user1"
      }
      at_least_one_of :name
    end

    get :user do
      user = ::User.find_by(name: params["name"]) || "Can't find #{params["name"]}"
      present :user, user
    end
  end
end
```

# Demo

* [demo_rails_grape](https://github.com/mgleon08/demo_rails_grape)

Reference

* [grape](https://github.com/ruby-grape/grape)
* [grape-swagger](https://github.com/ruby-grape/grape-swagger)
* [grape-swagger-rails](https://github.com/ruby-grape/grape-swagger-rails)
* [rails + grape 快速API簡單搭建](https://pathbox.github.io/2016/05/28/rails-grape-api/)
* [在rails中使用grape來建立API](https://sibevin.github.io/posts/2015-10-01-165320-create-api-with-grape-in-rails)
