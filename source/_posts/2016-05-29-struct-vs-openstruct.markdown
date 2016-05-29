---
layout: post
title: "Struct vs OpenStruct"
date: 2016-05-29 20:18:53 +0800
comments: true
categories: ruby
---

在 ruby 當中，經常會要定義一個新的類別，如果覺得每次都要寫 `class xxx` 太麻煩，就可以用 struct & OpenStruct 快速的產生出來!

<!-- more -->

#Class
```ruby
class People
  attr_accessor :name, :phone

  def initialize(name, phone)
    @name  = name
    @phone = phone
  end

  def to_ary
    [name, phone]
  end
end

a = People.new("foo", 1234)
#=> #<People:0x007fcaabcf5328 @name="foo", @phone=1234>
a.name
#=> "foo"
a.phone
#=> 1234
a.to_ary
#=> ["foo", 1234]
```

#Struct

```ruby
#method 要用 block 來傳遞，Attribute 一開始就固定了
People = Struct.new(:name, :phone) do
  def to_ary
    [name, phone]
  end
end
# => People

a = People.new("foo", 1234)
#=> #<struct People name="foo", phone=1234>
a.name
#=> "foo"
a.phone
#=> 1234
a.to_ary
#=> ["foo", 1234]
```
###其他取 Attribute Value 的方法
Class則無法

```ruby
a[:name]
#=> "foo"
a["name"]
#=> "foo"
a[0]
#=> "foo"
```
#OpenStruct
主要差異點是在於，比 Struct 更有彈性, 因為它可以任意增加 Attribute , 不像 Struct 要先限制好有哪些 Attribute

但比較可惜的是，無法定義 method

```ruby
#在 console 記得先 require
require 'ostruct'

People = OpenStruct.new
#=> #<OpenStruct>
or
People = OpenStruct.new(name: 'foo', phone: 1234)
#=> #<OpenStruct name="foo", phone=1234>

可以自由新增
People.name = 'foo'
#=> "foo"
People.phone = 1234
#=> 1234
People.age = 18
#=> 18
People
#=> #<OpenStruct name="foo", phone=1234, age=18>
```

###WHEN TO USE?

* As a temporary data structure 暫時的 data 結構
* As internal class data 內部的 class data

>也許另一個 class 還不至於明確到可以獨立成一個 class，因此先暫存在別的 class 裡，直到有明確的行為，足夠讓它獨立出去

```ruby
class Person
  Address = Struct.new(:street_1, :street_2, :city)

  attr_accessor :name, :address

  def initialize(name, opts)
    @name = name
    @address = Address.new(opts[:street_1], opts[:street_2], opts[:city])
  end
end

leigh = Person.new("Leigh Halliday", {
  street_1: "123 Road",
  city: "Toronto",
})

puts leigh.address.inspect
# <struct Person::Address street_1="123 Road", street_2=nil, city="Toronto", province="Ontario", country="Canada", postal_code="M5E 0A3">
```

* As a testing stub

```ruby
KCup = Struct.new(:size, :brewing_time, :brewing_temp)
colombian = KCup.new(:small, 60, 85)

brewer = Brewer.new(colombian)
expect(brewer.brew).to eq(true)
```

官方文件：  
[Struct](http://ruby-doc.org/core-2.2.0/Struct.html)  
[OpenStruct](http://ruby-doc.org/stdlib-2.0.0/libdoc/ostruct/rdoc/OpenStruct.html)

參考文件：  
[When should I use Struct vs. OpenStruct?](http://stackoverflow.com/questions/1177594/when-should-i-use-struct-vs-openstruct#answer-4459132)  
[The simple but powerful Ruby Struct](https://www.leighhalliday.com/ruby-struct)  
[模擬class物件：Ruby當中Struct及OpenStruct的使用](http://motion-express.com/blog/20150406-ruby-struct-and-ostruct)  