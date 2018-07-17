---
layout: post
title: "rails 中的 attr_accessor, has_many, scope 怎麼來?"
date: 2018-07-04 21:56:05 +0800
comments: true
categories: rails
---

相信學 rails 的一定對 `attr_accessor`, `has_many`, `scope` 這些不陌生

<!-- more -->

但是到底是為什麼可以在 model 裡面直接下這些 method?

先看一下 rails 的 model，可以這樣定義

```ruby
# 5.x.x
class Klass < ApplicationRecord
  attr_accessor :name
  has_many :books
  scope :male, ->{ where(gender:1) }
end

# 4.x.x
class Klass < ActiveRecord::Base
  attr_accessor :name
  has_many :books
  scope :male, ->{ where(gender:1) }
end
```

可以知道 model 裡的是 `class` 並且是繼承 `ApplicationRecord` 因此判斷可能是屬於 `ApplicationRecord` 或者是更上層的某一個物件

在 ruby 中，所有類別的類別都是 class 類別

```ruby
Object.class
# => Class
Kernel.class
# => Module
Module.class
# => Class
Class.class
# => Class
```

### Class 的寫法

一般看到的 class

```ruby
class Klass
  def hi
    puts 'hi'
  end
end

```

相當於

```ruby
# 所有類別的名字都是一個常數(大寫開頭)，如果不是大寫就不會有自己的名字
Klass = Class.new do
  def hi
    puts 'hi'
  end
end
```

既然知道了 model 裡的 class 實際上是 `Class.new` 接下來看一下 doc [Class new](https://ruby-doc.org/core-2.2.0/Class.html#method-c-new)

> If a block is given, it is passed the class object, and the block is evaluated in the context of this class using class_eval.

意思是指，中間那段 Block 都會用 `class_eval` 來執行，


```ruby
Klass = Class.new do
  def meth1
    "hello"
  end
  def meth2
    "bye"
  end
end

a = Klass.new     #=> #<Klass:0x007f916006aa60>
a.meth1          #=> "hello"
a.meth2          #=> "bye"
```

由此推斷以下相等

```ruby
class Klass
  attr_accessor :name
  has_many :books
  scope :male, ->{ where(gender:1) }
end

Klass = Class.new do
  attr_accessor :name
  has_many :books
  scope :male, ->{ where(gender:1) }
end

class Klass; end
Klass.class_eval do 
  attr_accessor :name
  has_many :books
  scope :male, ->{ where(gender:1) }
end
```

由此可知，`attr_accessor`, `has_many`, `scope` 都是由上層物件所定義好的 method

看一下原始碼其中一段，可看到 `has_many`，是在一個 module 裡面，應該是透過 `include` 或是 `extend` 的方式，變成了 class 的 methods

```ruby
# rails/activerecord/lib/active_record/associations.rb
# ...
module ClassMethods
# ...
  def has_many(name, scope = nil, **options, &extension)
    reflection = Builder::HasMany.build(self, name, scope, options, &extension)
  Reflection.add_reflection self, name, reflection
  end
# ...
end
# ...
```

主要是研究一下看 rails 一些特別的功能是怎麼來的，要仔細的話就要去挖原始碼才能知道了

參考文件

* [用 Instance_eval & Class_eval 自己加 Method!](http://mgleon08.github.io/blog/2016/03/08/instance-eval-class-eval/)
* [Ruby 中的 Include Extend Require Load](http://mgleon08.github.io/blog/2016/02/24/include-extend-require/)
* [自由的 Ruby 類別（一）](https://blog.frost.tw/posts/2017/10/22/The-ruby-s-class-is-free-Part-1/)
