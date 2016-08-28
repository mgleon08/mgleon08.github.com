---
layout: post
title: "更改 Rails 命名規則 Inflector"
date: 2016-08-11 20:49:39 +0800
comments: true
categories: rails
---

在 rails 當中，有許多命名的慣例

<!-- more -->

* 像是 `hello_controller` 檔案，裡面慣例對應到的 Class 就是 `Hello_Controller`
* 但若是像 `api_controller`，就會是 `Api_Controller`

雖然不是不行，但 API 本來就是縮寫，硬要改成 Api 可能會讓人覺得怪怪的


因此 rails 有提供解決辦法，就能夠將 `api_Controller` 改成 `API_Controller`

rails 內建有 `Inflector` 就是用來處理命名規則用的

```ruby
config/initializers/inflections.rb

ActiveSupport::Inflector.inflections(:en) do |inflect|
  inflect.acronym 'API'
end
```

或是 Rails 轉單複數單字，有錯誤時，可以定義

```ruby
#錯誤
"Business".singularize  => "Busines"
"moose".pluralize => "mooses"
```

```ruby
config/initializers/inflections.rb

ActiveSupport::Inflector.inflections(:en) do |inflect|
  inflect.irregular 'tooth', 'teeth'
end
```

###other

```ruby
# Be sure to restart your server when you modify this file.

# Add new inflection rules using the following format. Inflections
# are locale specific, and you may define rules for as many different
# locales as you wish. All of these examples are active by default:
# ActiveSupport::Inflector.inflections(:en) do |inflect|
#   inflect.plural /^(ox)$/i, '\1en'
#   inflect.singular /^(ox)en/i, '\1'
#   inflect.irregular 'person', 'people'
#   inflect.uncountable %w( fish sheep )
# end

# These inflection rules are supported but not enabled by default:
# ActiveSupport::Inflector.inflections(:en) do |inflect|
#   inflect.acronym 'RESTful'
# end
```

官方文件：

* [Inflector](http://api.rubyonrails.org/classes/ActiveSupport/Inflector.html)

參考文件：

* ['APIController' is not a supported controller name](https://ruby-china.org/topics/19242)
* [環境設定與Bundler](https://ihower.tw/rails4/environments-and-bundler.html)