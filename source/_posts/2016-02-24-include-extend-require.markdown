---
layout: post
title: "Ruby 中的 include extend require"
date: 2016-02-24 21:49:30 +0800
comments: true
categories: ruby
---

常常搞不清楚，`include`，`extend`，`require`這幾個差異。

<!-- more -->

# module
`include` & `extend` 主要是將 `module` 的方法，可以繼承給 `class` 或 `instance` 使用。

```ruby
module Foo   
  def hello     
    puts 'Hello!'   
  end 
end

class Bar
  include Foo
end

Bar.hello
# => `<main>': undefined method `hello' for #<Bar:0x007fd815939618> (NoMethodError)
a = Bar.new
a.hello
# => Hello!

class Bar
  extend Foo
end

Bar.hello
# => Hello!
a = Bar.new
a.hello
# => `<main>': undefined method `hello' for #<Bar:0x007fd815939618> (NoMethodError)
```
由此可知 

* `extend`  增加 class_methods
* `include` 增加 instance_methods

#require

在 ruby 中，一開始用有很多 `method` 可以使用，是因為 ruby 將一些常用的都先載入進來。
  
* [core](http://ruby-doc.org/core-2.3.0/)

而其他比較不常用的 method 就必須在要使用的時候，先 require 進來才可以。  

* [std](http://ruby-doc.org/stdlib-2.3.0/)

```ruby
require 'fileutils'
```

也可以載入自己設定好的檔案，不過記得路徑的問題，可以使用 `require_relative `

```ruby
require_relative 'test'
```

官方文件：  
[ruby-doc](http://ruby-doc.org/core-2.3.0/)  

參考文件：  
[Ruby 語法放大鏡之「類別跟模組有什麼不一樣?」](http://blog.eddie.com.tw/2015/03/24/class-and-module/)   
[require、require_relative是什麼意思？差在哪？](http://motion-express.com/blog/20150407-ruby-require-require-relative-load)