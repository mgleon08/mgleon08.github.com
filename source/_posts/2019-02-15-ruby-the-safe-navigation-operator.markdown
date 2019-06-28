---
layout: post
title: "Ruby - (&.) The Safe Navigation Operator like Rails try!"
date: 2019-02-15 20:46:20 +0800
comments: true
categories: ruby
---

<!-- more -->

`ruby 2.3.0` 之後有提供一個方法 `&.` 跟 rails 的 try! 一樣（是有 `!` 驚嘆號的）

`try!` 跟 `try` 比起來比較嚴謹，會去 check receiver 是否為 `nil`，如果都用只用 `try`，反而會導致都回傳 `nil` 而發生錯誤時也不知道在哪 

```ruby
def try(*a, &b)
  if a.empty? && block_given?
    yield self
  else
    __send__(*a, &b)
  end
end
```

```ruby
def try!(*a, &b)
  if a.empty? && block_given?
    yield self
  else
    public_send(*a, &b)
  end
end
```

# nil

rails 5.2.3

```ruby
class Account
  attr_reader :owner
  def initialize(owner)
    @owner = owner
  end
end

user = Account.new(nil)
user.owner.address
#NoMethodError (undefined method `address' for nil:NilClass)
user && user.owner && user.owner.address
# => nil
user.try(:owner).try(:address)
# => nil
user.try!(:owner).try!(:address)
# => nil
user&.owner&.address
# => nil
```

ruby 2.6.1

```ruby
class Account
  attr_reader :owner
  def initialize(owner)
    @owner = owner
  end
end

user = Account.new(nil)
user.owner.address
# NoMethodError (undefined method `address' for nil:NilClass)
user && user.owner && user.owner.address
# => nil
user.try(:owner).try(:address)
# => undefined method `try' for #<Account:0x00007fa43d0e25e8 @owner=nil>
user.try!(:owner).try!(:address)
# => NoMethodError (undefined method `try!' for #<Account:0x00007fa43d0e25e8 @owner=nil>)
user&.owner&.address
# => nil
```

# false

rails 5.2.3

```ruby
class Account
  attr_reader :owner
  def initialize(owner)
    @owner = owner
  end
end

user = Account.new(false)
user.owner.address
# NoMethodError (undefined method `address' for false:FalseClass)
user && user.owner && user.owner.address
# => false
user.try(:owner).try(:address)
# => nil
user.try!(:owner).try!(:address)
# NoMethodError (undefined method `address' for false:FalseClass)
user&.owner&.address
# NoMethodError (undefined method `address' for false:FalseClass)
```

ruby 2.6.1

```ruby
class Account
  attr_reader :owner
  def initialize(owner)
    @owner = owner
  end
end

user = Account.new(false)
user.owner.address
# NoMethodError (undefined method `address' for false:FalseClass)
user && user.owner && user.owner.address
# => false
user.try(:owner).try(:address)
# => NoMethodError (undefined method `try' for #<Account:0x00007fd8c78c9b08 @owner=false>)
user.try!(:owner).try!(:address)
# NoMethodError (undefined method `address' for false:FalseClass)
user&.owner&.address
# NoMethodError (undefined method `address' for false:FalseClass)
```

# Object.new

rails 5.2.3

```ruby
class Account
  attr_reader :owner
  def initialize(owner)
    @owner = owner
  end
end

user = Account.new(Object.new)
user.owner.address
# NoMethodError (undefined method `address' for #<Object:0x00007fcb887785e8>)
user && user.owner && user.owner.address
# NoMethodError (undefined method `address' for #<Object:0x00007fcb887785e8>)
user.try(:owner).try(:address)
# => nil
user.try!(:owner).try!(:address)
# NoMethodError (undefined method `try!' for #<Account:0x00007fd8c7893800 @owner=#<Object:0x00007fd8c7893828>>)
user&.owner&.address
# NoMethodError (undefined method `address' for #<Object:0x00007fd8c7893828>)
```

ruby 2.6.1

```ruby
class Account
  attr_reader :owner
  def initialize(owner)
    @owner = owner
  end
end

user = Account.new(Object.new)
user.owner.address
# NoMethodError (undefined method `address' for #<Object:0x00007fcb887785e8>)
user && user.owner && user.owner.address
# NoMethodError (undefined method `address' for #<Object:0x00007fcb887785e8>)
user.try(:owner).try(:address)
# => NoMethodError (undefined method `try' for #<Account:0x00007fd8c7893800 @owner=#<Object:0x00007fd8c7893828>>)
user.try!(:owner).try!(:address)
# NoMethodError (undefined method `try!' for #<Account:0x00007fd8c7893800 @owner=#<Object:0x00007fd8c7893828>>)
user&.owner&.address
# NoMethodError (undefined method `address' for #<Object:0x00007fd8c7893828>)
```

參考文件:

* [The Safe Navigation Operator (&.) in Ruby](http://mitrev.net/ruby/2015/11/13/the-operator-in-ruby/)
* [try](https://apidock.com/rails/v5.2.3/Object/try)
* [try!](https://apidock.com/rails/v5.2.3/Object/try%21)
