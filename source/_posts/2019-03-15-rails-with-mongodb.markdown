---
layout: post
title: "Rails with mongoDB"
date: 2019-03-15 23:01:55 +0800
comments: true
categories: rails
---

<!-- more -->

> Version: Ruby 2.5.3, Rails 5.2.2

# Generate new project

```ruby
rails new demo_rails_mongodb --skip-active-record --api -C
```

# Add mongoid gem

* [mongoid](https://github.com/mongodb/mongoid)

```ruby
gem 'mongoid', '~> 7.0.2'
```

create `config/mongoid.yml`

```ruby
rails g mongoid:config
```

# Add docker-compose

```ruby
# docker-compose.yml
version: '3'
services:
  mongo:
    image: mongo:4.1
    container_name: mongo4
    restart: always
    ports:
      - '27017:27017'
    volumes:
      - ./tmp/data/mongo/data:/data/db
```

```ruby
docker-compose up
```

# Create model & data

### Scaffold

```ruby
rails generate scaffold book title:string author:string pages:integer
```

```ruby
# rails c
Book.create(title: "First", pages:20, author:"leon")
```

# View

```ruby
# rails s -p 4321
http://localhost:4321/books
```

# Demo

* [demo_rails_mongodb](https://github.com/mgleon08/demo_rails_mongodb)

Reference

* [Mongoid](https://docs.mongodb.com/mongoid/current/)
* [old mongoid doc](https://mongoid.github.io/old/en/mongoid/index.html)
* [MongoDB 基礎入門教學：MongoDB Shell 篇](https://blog.gtwang.org/programming/getting-started-with-mongodb-shell-1/)
