---
layout: post
title: "Ruby Tips - alias vs alias method "
date: 2019-02-04 17:38:58 +0800
comments: true
categories: ruby ruby_tips
---

<!-- more -->

# alias

* 建立 method 的別名
* 可以別名全域變數
* 在 `libdoc` 之下的 `RDoc` 裡的關鍵字
* 可使用 `method name` 或 `symbol`
* scope 只在其關鍵字存在的scope

```ruby
class User
  def hi
    'hi'
  end

  alias hello hi
end

user = User.new # => #<User:0x007fedc406afe0>
user.hi         # => "hi"
user.hello      # => "hi"
```

另外也可以別名全域變數

```ruby
$a = 1
alias $b $a
$b
# => 1
```
scope

```ruby
class Animal
  def say
    'hi'
  end

  def self.add_hi
    alias :hi :say 
    # scope 只在 Animal，因此會是 hi
  end
end

class Dog < Animal
  def say
    'hello'
  end
  add_hi
end

Dog.new.hi # => "hi"
```

# alias_method

> Makes new_name a new copy of the method old_name. This can be used to retain access to methods that are overridden.

* 是 module 類別的一個方法
* 可以是 `string` 或 `symbol`
* 要有 `,` 區隔
* scope 可以到父類別繼承下來的 method

```ruby
class User
  def hi
    'hi'
  end

  alias_method :hello, :hi
end

user = User.new # => #<User:0x007fcb8e0c8a08>
user.hi         # => "hi"
user.hello      # => "hi"
```

將原本的 `exit` 別名為 `orig_exit`，再將原本的 `exit` method 覆蓋

```ruby
module Mod
  alias_method :orig_exit, :exit
  def exit(code=0)
    puts "Exiting with code #{code}"
    orig_exit(code)
  end
end
include Mod
exit(99)
```

scope 在 當前

```ruby
class Animal
  def say
    'hi'
  end

  def self.add_hi
    alias_method :hi, :say
    # scope 到繼承類別，say 被 Dog 覆蓋掉，因此會是 hello
  end
end

class Dog < Animal
  def say
    'hello'
  end
  add_hi
end

Dog.new.hi # => "hello"
```

參考文件

* [alias](http://ruby-doc.org/stdlib-2.5.1/libdoc/rdoc/rdoc/RDoc/Alias.html)
* [alias_method](https://ruby-doc.org/core-2.2.2/Module.html#method-i-alias_method)
* [ruby 中 alias vs alias_method](https://www.jianshu.com/p/cebbdf6d5672)
