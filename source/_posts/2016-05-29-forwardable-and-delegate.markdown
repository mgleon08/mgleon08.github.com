---
layout: post
title: "Forwardable 轉發 & Delegate 委派"
date: 2016-05-29 20:15:26 +0800
comments: true
categories: ruby
---

在 OO 的世界裡，經常會去用到不同 class 的 method，因此透過這個方法，可以將不同 class 的 method `轉發` 過來!

<!-- more -->

#Forwardable

顧名思義，將訊息 `轉發` 給別的物件。

```ruby
Account = Struct.new(:first_name, :last_name)

class User
  attr_reader :account

  def initialize(account)
    @account = account
  end

  def first_name
    account.first_name
  end

  def last_name
    account.last_name
  end

  def full_name
    "#{first_name} #{last_name}"
  end
end

user = User.new(Account.new('foo', 'bar'))

puts user.first_name
puts user.last_name
puts user.full_name
#=>foo
#=>bar
#=>foo bar
```

上述有許多重複的 account，此時就可以使用 Forwardable 來簡化：  
將 first_name、last_name `轉發` 給 account。

```ruby
require 'forwardable'

Account = Struct.new(:first_name, :last_name)

class User
  attr_reader :account
  extend Forwardable
  def_delegators :account, :first_name, :last_name
 	
  #若 User 不想對外開放，attr_reader :account，可以改成實例變數，如以下
  #extend Forwardable
  #def_delegators :@account, :first_name, :last_name
  
  def initialize(account)
    @account = account
  end

  def full_name
    "#{first_name} #{last_name}"
  end
end

user = User.new(Account.new('foo', 'bar'))

puts user.first_name
puts user.last_name
puts user.full_name
#=>foo
#=>bar
#=>foo bar
```

###def_delegator(accessor, method, ali = method)
 
 一次只能 '轉發' 一個方法，第三個參數是（可選的）別名。
 
 ```ruby
class MyQueue
  extend Forwardable
  attr_reader :queue
  def initialize
    @queue = []
  end

  def_delegator :@queue, :push, :mypush
end

q = MyQueue.new
q.mypush 42
q.queue    #=> [42]
q.push 23  #=> NoMethodError
 ```


###def_delegators(accessor, *methods)

一次可以 '轉發' 多個方法。

```ruby
def_delegators :@account, :first_name, :last_name
```

###delegate [method, method, ...] => accessor

接受Hash

```ruby
delegate [:first_name, :last_name] => :account
```

看是如何 work

```ruby
module Forwardable
  def delegate(hash)
    hash.each{ |methods, accessor|
      methods.each{ |method|
        instance_eval %{
          def #{method}(*args, &block)
            #{accessor}.__send__(:#{method}, *args, &block)
          end
        }
      }
    }
  end
end
```
#Other
* `instance_delegate` alias `delegate`
* `def_instance_delegator` alias `def_delegator`
* `def_instance_delegators` alias `def_delegators` 

* [rails-delegate](http://mgleon08.github.io/blog/2015/12/13/ruby-on-rails-delegate/) 之前就有寫到，可以參考!
 
 
官方文件：  
[instance_delegate](http://apidock.com/ruby/Forwardable/instance_delegate)  
[def_instance_delegator](http://apidock.com/ruby/Forwardable/def_instance_delegator)  
[def_instance_delegators](http://apidock.com/ruby/Forwardable/def_instance_delegators)  
[Forwardable](http://apidock.com/ruby/Forwardable)  
[Forwardable](http://ruby-doc.org/stdlib-2.0.0/libdoc/forwardable/rdoc/Forwardable.html)

參考文件：  
[Ruby 標準函式庫 Forwardable](http://juanitofatas.com/2014/06/30/ruby-stdlib-forwardable/)  
[深入理解和学习Ruby的Forwardable](http://qiita.com/xiangzhuyuan/items/409c87da8cc4a882419b)  
[Delegating All of the Things With Ruby Forwardable](http://vaidehijoshi.github.io/blog/2015/03/31/delegating-all-of-the-things-with-ruby-forwardable/)