---
layout: post
title: "Ruby 中的 include extend require load"
date: 2016-02-24 21:49:30 +0800
comments: true
categories: ruby
---

常常搞不清楚，`include`，`extend`，`require`，`load`這幾個差異。

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


###也可以用 include 導入 module 的 method

`module.rb`

```ruby
module Foo
  def bar(a)
    puts "#{a}"
  end
end
```

`test.rb`

```ruby
require './module'
include Foo

bar('hello')
or 
Foo.bar('hello')
```

#require

在 ruby 中，一開始用有很多 `method` 可以使用，是因為 ruby 將一些常用的都先載入進來。
  
* [core](http://ruby-doc.org/core-2.3.0/)

而其他比較不常用的 method 就必須在要使用的時候，先 require 進來才可以。  

* [std](http://ruby-doc.org/stdlib-2.3.0/)

```ruby
require 'fileutils'
```

也可以載入自己設定好的檔案，通常設定在 `/lib/test.rb`  

```ruby
require 'test'
```

不過記得路徑的問題，可以使用 `require_relative `

```ruby
require_relative 'test'
```

如果是放在資料夾底下的話 `/lib/file/test.rb` 

```ruby
require 'file/test'
```

#Load

跟 require 類似，不過每 load 一次，就會重新執行該 load 的檔案。

>以術語來說，load 要求載入一個檔案，而require要求某個功能特性（Feature）

```ruby
load "sample.rb" #load 要加 .rb
```


官方文件：  
[ruby-doc](http://ruby-doc.org/core-2.3.0/)  

參考文件：  
[Ruby 語法放大鏡之「類別跟模組有什麼不一樣?」](http://blog.eddie.com.tw/2015/03/24/class-and-module/)   
[require、require_relative是什麼意思？差在哪？](http://motion-express.com/blog/20150407-ruby-require-require-relative-load)  
[環境設定與Bundle](https://ihower.tw/rails4/environments-and-bundler.html)  
[load 與 require](http://openhome.cc/Gossip/Ruby/LoadRequire.html)