---
layout: post
title: "Ruby - Dry your code with tap"
date: 2019-02-04 17:46:18 +0800
comments: true
categories: ruby
---

<!-- more -->

> Yields self to the block, and then returns self. The primary purpose of this method is to “tap into” a method chain, in order to perform operations on intermediate results within the chain.

在 ruby 中，因為太過方便很常會用到 `method chain`，透過 `tap` 可以變得更 dry

實際上就是，透過 `yield` 將 block 帶過去，再透過 self 將自身帶回來

```ruby
# activesupport/lib/active_support/core_ext/object/misc.rb, line 53
def tap
  yield self
  self
end
```

另一個用處，便於檢查 `method chain` 裡的 info

```ruby
(1..10)                .tap {|x| puts "original: #{x.inspect}"}
  .to_a                .tap {|x| puts "array: #{x.inspect}"}
  .select {|x| x%2==0} .tap {|x| puts "evens: #{x.inspect}"}
  .map { |x| x*x }     .tap {|x| puts "squares: #{x.inspect}"}

# original: 1..10
# array: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
# evens: [2, 4, 6, 8, 10]
# squares: [4, 16, 36, 64, 100]
```

```ruby
class Dog
  attr_accessor :name

  def self.create_instance
    dog = new
    dog.name = "hey"
    dog
  end
end

dog = Dog.create_instance
dog.name
```

透過 `tap` 改成

```ruby
class Dog
  attr_accessor :name

  def self.create_instance
    Dog.new.tap do |dog|
      dog.name = "hey"
    end
  end
end

dog = Dog.create_instance
dog.name
```

參考文件

* [ruby doc tap](https://ruby-doc.org/core-2.6.1/Object.html#method-i-tap)
