---
layout: post
title: "Ruby - extend vs include (Mixin) "
date: 2019-02-04 17:23:28 +0800
comments: true
categories: ruby
---

<!-- more -->

* `extend`: 讓 module 內定義的 method 成為 instance method
* `include`: 讓 module 內定義的 method 成為 class method

```ruby
module Animal
  def say
    'hi'
  end
end

class Dog
  include Animal
  def initialize
  end
end

Dog.say
#NoMethodError: undefined method `say' for Dog:Class
Dog.new.say
# => "hi"

class Cat
  extend Animal
end

Cat.say
# => "hi"
Cat.new.say
#NoMethodError: undefined method `say' for #<Cat:0x007fc7e781efa0>

# module 本身也是 class (module.class) 因此可以透過 extend 來延展
module Pig
  extend Animal
end

Pig.say
# => "hi"
Pig.new.say
# NoMethodError: undefined method `new' for Pig:Module
```

參考文件

* [Ruby 中的 Include Extend Require Load]({{ root }}/blog/2016/02/24/include-extend-require/)
