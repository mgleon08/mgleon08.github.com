---
layout: post
title: "Ruby - ERB template"
date: 2016-08-30 10:31:47 +0800
comments: true
categories: ruby
---

相信學過 rails 一定知道 erb，可以很方便地將 ruby code 寫在 view 中

<!-- more -->

但不一定是要有 rails 才可以用 erb

```ruby
#template.rb
require 'erb'

class Template
  def initialize(foo, bar)
    @foo = foo
    @bar = bar
  end

  def open_template
    puts ERB.new(File.read(templates_path), nil).result(binding)
  end

  def templates_path
    File.expand_path("../template.erb", __FILE__)
  end
end

Template.new('hi', 'hello').open_template
```

```ruby
#template.erb
1. <%= @foo %>
2. <%= @bar %>
```

```ruby
#執行
ruby template.rb
#1. hi
#2. hello
```
### 也可以利用 erb 來寫設定檔

* [用 Yaml 來寫文件, 設定檔](http://mgleon08.github.io/blog/2016/02/07/yaml/)

參考文件：

* [ERB doc](http://ruby-doc.org/stdlib-2.3.1/libdoc/erb/rdoc/ERB.html)
* [Ruby templates: How to pass variables into inlined ERB?](http://stackoverflow.com/questions/1338960/ruby-templates-how-to-pass-variables-into-inlined-erb)
* [Render an ERB template with values from a hash](http://stackoverflow.com/questions/8954706/render-an-erb-template-with-values-from-a-hash)
