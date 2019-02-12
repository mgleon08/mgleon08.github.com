---
layout: post
title: "Ruby - &: and &method with to_proc"
date: 2019-02-04 17:58:51 +0800
comments: true
categories: ruby
---

<!-- more -->

在 ruby 中很常看到以下的寫法，那為什麼可以簡寫成 `(&:to_s)`

```ruby
[1, 2, 3].map { |num| num.to_s } #=>  => ["1", "2", "3"]
[1, 2, 3].map(&:to_s) #=>  => ["1", "2", "3"]
```

`&` 實際上是會觸發物件的 `to_proc` 方法，把物件轉換為 `Proc`，並嘗試指定給 `&` ，因此可以在物件上定義 `to_proc`，然後使用 & 來觸發

可以看到，建立一個自己的 `to_proc` 把原本的覆蓋掉，並加上一些訊息

```ruby
class Symbol
  def to_proc
    puts "self is #{self}"
    Proc.new { |obj, *args| obj.send(self, *args) }
  end
end

[1, 2, 3].map(&:to_s)
# self is to_s
#  => ["1", "2", "3"]
```

symbol 都會有 `to_proc` method

```ruby
:symbol.methods
# => [:to_proc, ..]
:upcase.to_proc
# => #<Proc:0x007fb2410f3e30(&:upcase)>
:upcase.to_proc.call('abc')
# self is upcase
# => "ABC"
p = Proc.new{ |n| n ** 2 }
[1, 2, 3].map(&p)
# => [1, 4, 9]
```

但是在 `reduce/inject` 卻可以將 `&` 拿掉

```ruby
[1, 2, 3].reduce(&:+) # => 6
[1, 2, 3].reduce(:+)  # => 6
[1, 2, 3].reduce('+') # => 6

[1, 2, 3].inject(&:+) # => 6
[1, 2, 3].inject(:+)  # => 6
[1, 2, 3].inject('+') # => 6
```

# &method

> Looks up the named method as a receiver in obj, returning a Method object (or raising NameError). The Method object acts as a closure in obj's object instance, so instance variables and the value of self remain available.

如果要將原本的值變成參數，就要改用 `&method`

```ruby
[1, 2, 3].map { |num| puts(num) }
[1, 2, 3].map(&method(:puts))
```

參考文件

* [Ruby 魔法之 Symbol#to_proc() 方法](https://www.jianshu.com/p/4fa98d829fc9)
* [How Does Symbol#to_proc Work?](http://benjamintan.io/blog/2015/03/16/how-does-symbol-to_proc-work/)
* [What are :+ and &:+ in Ruby?](https://stackoverflow.com/questions/2697024/what-are-and-in-ruby/51572627)
* [What does map(&:name) mean in Ruby?](https://stackoverflow.com/questions/1217088/what-does-mapname-mean-in-ruby)
* [method](https://ruby-doc.org/core-2.6.1/Object.html#method-i-method)
