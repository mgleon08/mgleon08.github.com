---
layout: post
title: "用 instance_eval & class_eval 自己加 method! "
date: 2016-03-08 22:46:28 +0800
comments: true
categories: ruby
---

有時候會發現有些 object，沒有設定 setter & getter ，剛好又需要用到，這時就可以派上用場拉。

<!-- more -->

#instance_eval & class_eval

`instance_eval` Use ClassName.instance_eval to define a class method (one associated with the class object but not visible to instances).

`class_eval` Use ClassName.class_eval to define an instance method (one that applies to all of the instances of ClassName).

簡單的來說，就是自己定義 class or instance 的 method

```ruby
class MyClass
  def initialize(num)
    @num = num
  end
end

a = MyClass.new(1)
b = MyClass.new(2)
```

錯誤，因為沒有 getter or setter methods

```ruby
a.num
#=>NoMethodError: undefined method `num' for #<MyClass:0x007fba5c02c858 @num="1">
```

自行加 instance method

```ruby
a.instance_eval { @num }
#=> 1
```

也可以直接定義 method，這樣就可以一直重複使用  

```ruby
a.instance_eval do
   def num
     @num
   end
end

a.num
#=> 1
b.num
#NoMethodError: undefined method `num' for #<MyClass:0x007fba5c08e5f8 @num="2">
```

上述指定義了 a 的 `instance method` 所以 b 會錯誤。  
因此可以直接定義在 `class instance`


```ruby
MyClass.class_eval do
  def num
  	@num
  end
end

a.num
#=> 1
b.num
#=> 2
```

參考文件：  
[Understanding class_eval and instance_eval](http://web.stanford.edu/~ouster/cgi-bin/cs142-winter15/classEval.php)  
[eval](http://openhome.cc/Gossip/Ruby/Eval.html)