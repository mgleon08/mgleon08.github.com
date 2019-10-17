---
layout: post
title: "Ruby - public protected private"
date: 2019-02-04 17:12:51 +0800
comments: true
categories: ruby
---

<!-- more -->

* `public`: 不管在哪都可以 call
* `private`: 必須在類別內部才可以 call，不能明確指明 `recevier`，即使 self 也不行
* `protected`: 在 `public` 和 `private` 兩者之間，不能給外面呼叫，但在類別內，是可以自由取用(要不要加 recevier 都可以)

```ruby
class User
  def initialize
  end

  def public
    "public"
  end

  def call_protected
    protected
  end

  def call_private
    private
  end

  def call_protected_with_recevier
    self.protected
  end

  def call_private_with_recevier
    self.private
  end

  protected

  def protected
    "protected"
  end

  private

  def private
    "private"
  end

  # 另一種指定方式
  # protected :protected
  # private :private
end

user = User.new     # => #<User:0x007fa756804198>
user.public         # => "public"
user.protected      # NoMethodError: protected method `protected' called for #<User:0x007fa756804198>
user.private        # NoMethodError: private method `private' called for #<User:0x007fa756804198>
user.call_protected # => "protected"
user.call_private   # => "private"
user.call_protected_with_recevier # => "protected"
user.call_private_with_recevier   # NoMethodError: private method `private' called for #<User:0x007fa756804198>
```

另外有個例外，如果像上面的方式將 `class method` 放在 private 會無效

```ruby
class A
  private

  def self.b
    puts 'hi'
  end
end

A.b
# hi
# => nil
```

必須使用 `self` 或 `private_class_method`

> `class << self` opens up self's singleton class, so that methods can be redefined for the current self object. This is used to define class/module ("static") method. Only there, defining private methods really gives you private class methods.

```ruby
class A
  class << self
    private
    def b
      'hi'
    end
  end

  def self.c
    hello
  end
  private_class_method :c
end

A.b # NoMethodError: private method `b' called for A:Class
A.c # NoMethodError: private method `c' called for A:Class
```

一般都只會用 `private`，而用 `protected` 的情況如以下

```ruby
class User
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def ==(other_user)
    # 當兩個 instance 都要指名 receiver 時
    self.secret == other_user.secret
  end

  protected

  def secret
    "#{name}.#{name.length}"
  end
end

foo = User.new("Foo")
bar = User.new("Bar")
foo == bar # => false
```

參考文件

* [類別（Class）與模組（Module）](https://railsbook.tw/chapters/08-ruby-basic-4.html)
* [How to create a private class method?](https://stackoverflow.com/questions/4952980/how-to-create-a-private-class-method)
* [Public, Private and Protected methods in Ruby](http://rubyblog.pro/2016/11/public-protected-private-in-ruby)
