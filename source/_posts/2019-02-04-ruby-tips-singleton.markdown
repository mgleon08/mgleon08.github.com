---
layout: post
title: "Ruby Tips - singleton method vs singleton class vs singleton module"
date: 2019-02-04 18:15:16 +0800
comments: true
categories: ruby ruby_tips
---

<!-- more -->

* [singleton method](#method)
* [singleton class](#class)
* [singleton module](#module)

# <span id="method"> singleton method </span>

* `singleton method`: 屬於某一個 object 的方法，也代表只屬於該 object，儘管有相同的 class 也無法使用別人的 `singleton method` (但 `extend` 可以)

```ruby
class User
  def self.class_hi
    'class_hi'
  end

  class << self
    def class_hi_2
      'class_hi_2'
    end
  end

  def instance_hi
    'instance_hi'
  end
end

def User.class_hi_3
  'class_hi_3'
end
```

因為 `User class` 所定義的 `class_method` 都屬於 `User class` object

```ruby
User.singleton_methods # => [:class_hi, :class_hi_2, :class_hi_3]
```

每次 new 的 instance 都會有 instance_hi 所以不算是 singleton_method，並沒有屬於哪一個 object

```ruby
User.new.singleton_methods # => []
leon = User.new
leon.singleton_methods # => []
```

建立專屬於 leon object 的 method

```ruby
def leon.hello
  'hello'
end 

class << leon
  def foo
    "Hello World!"
  end
end

leon.singleton_methods # => [:hello, : foo]
```

leon object 所定義的 method 並不存在於 `User class`，而是在 leon object 後面的 `singleton_class`

```ruby
User.method_defined?(:instance_hi) # => true
User.method_defined?(:hello) # => false
leon.singleton_class.method_defined?(:instance_hi) # => true
leon.singleton_class.method_defined?(:hello) # => true
User.new.singleton_class.method_defined?(:instance_hi) # => true
User.new.singleton_class.method_defined?(:hello) #=> false
```

覆蓋原本 instance 的 method

```ruby
leon.instance_hi # => "instance_hi"

def leon.instance_hi
  'singleton_hi'
end 

leon.instance_hi # => "singleton_hi"
```

instance_eval 是所有 instance 都會有，所以也不是 singleton_method

```ruby
User.instance_eval do
   def test
     'instance_eval'
   end
end

User.new.singleton_methods # => []
```

透過 `singleton_method` 取得 proc 在用 call 呼叫

```ruby
hi = leon.singleton_method(:instance_hi)
hi.call
```

# <span id="class"> singleton class </span>

* `singleton class`: 一個隱藏在物件（不管是普通物件還是類）後面的一個特殊類，它只有一個實例（就是它自己）

![](https://www.devalot.com/assets/articles/2008/09/ruby-singleton/singleton-array.jpg)

> When you add a method to a specific object Ruby inserts a new anonymous class into the inheritance hierarchy as a container to hold these types of methods.

```ruby
leon.singleton_class
# => #<Class:#<User:0x007fa99c887c58>>
User.singleton_class
# => #<Class:User>

# #<Class: 開頭的都是 singleton_class
User.singleton_class.ancestors
# => [#<Class:User>, #<Class:Object>, #<Class:BasicObject>, Class, Module, Object, Kernel, BasicObject]
leon.singleton_class.singleton_methods
# => [:class_hi, :class_hi_2, :class_hi_3]
User.singleton_class?
# => false
User.singleton_class.singleton_class?
# => true
```

# <span id="module"> singleton module </span>
先來說一下 `singleton pattern`

在 `class` 當中，可以一直建立 `instance (object_id 都不同)`，每個 `instance` 都會佔用 memory，但有時候只需要建立一個，但為了避免建立多個，而造成 memory 的浪費，於是就有了 `singleton module` 

在 ruby 當中不是 core library 所以必須引入 `singleton`，可以發現當 `include Singleton` 後 `User.new` 就不能使用了

```ruby
require 'singleton'

class User
  include Singleton
  
  attr_accessor :name, :phone
end

User.instance # => #<User:0x007f951a821ed0>
User.new # => NoMethodError: private method `new' called for User:Class
```

可以發現，每次的 `instance` 其實都是同一個 object(這樣就不會浪費 memory)

```ruby
User.instance.object_id
# => 70139185794920
User.instance.object_id
# => 70139185794920
User.instance.object_id
# => 70139185794920
```

原本的 `new` 則是變成了 `private method`，而不是不見了
 
```ruby
 User.send(:new).object_id
# => 70139173678780
User.send(:new).object_id
# => 70139173207900
User.send(:new).object_id
# => 70139177568960
```

透過 [ObjectSpace](https://ruby-doc.org/core-2.2.0/ObjectSpace.html#method-c-each_object) 來看一下所有存在的 object

> The ObjectSpace module contains a number of routines that interact with the garbage collection facility and allow you to traverse all living objects with an iterator.

```ruby
require 'singleton'

class User
  include Singleton
  
  attr_accessor :name, :phone
end

ObjectSpace.each_object(User){} # => 0
User.instance # => #<User:0x007fa18f02e008>
ObjectSpace.each_object(User){} # => 1
User.instance # => #<User:0x007fa18f02e008>
ObjectSpace.each_object(User){} # => 1
User.send(:new) #<User:0x007fa18f185cd0>
ObjectSpace.each_object(User){} # => 2
```

singleton 行為一樣會繼承 [使用範例](https://gist.github.com/mehdi-farsi/135d516254ae690335da0b14c13ed83b#file-singleton2_03-rb)

```ruby
require 'singleton'

class User
  include Singleton
  
  attr_accessor :name, :phone
end

class Worker < User
end

Worker.new # => NoMethodError: private method `new' called for Worker:Class
worker = Worker.instance # => #<Worker:0x007fb663837158>
worker.name = 'name' # => "name"
worker.phone = 'phone' # => "phone"

user = User.instance # => #<User:0x007fa673119790>
user.name # => nil
user.phone # => nil
```

```ruby
worker.clone # => TypeError: can't clone instance of singleton Worker
worker.dup # => TypeError: can't dup instance of singleton Worker
```

參考文件

* [Singleton](https://ruby-doc.org/stdlib-2.5.1/libdoc/singleton/rdoc/Singleton.html)
* [singleton_method](https://ruby-doc.org/core-2.5.1/Object.html#method-i-singleton_method)
* [Ruby-Basics Singleton Methods](https://bparanj.gitbooks.io/ruby-basics/content/sixth_chapter.html)
* [Ruby的Class與Eigenclass](https://medium.com/@zneuray/ruby%E7%9A%84class%E8%88%87eigenclass-f994aa2b988f)
* [What exactly is the singleton class in ruby?](https://stackoverflow.com/questions/212407/what-exactly-is-the-singleton-class-in-ruby)
* [Understanding Ruby Singleton Classes](https://www.devalot.com/articles/2008/09/ruby-singleton)
* [求解 singleton_class, singleton_methods 的深入問題](https://ruby-china.org/topics/31734)
* [The Singleton module in Ruby - Part I](https://medium.com/rubycademy/the-singleton-module-in-ruby-part-i-7a26de39319d)
* [The Singleton module in Ruby - Part II](https://medium.com/rubycademy/the-singleton-module-in-ruby-part-ii-91b74366dd00)
