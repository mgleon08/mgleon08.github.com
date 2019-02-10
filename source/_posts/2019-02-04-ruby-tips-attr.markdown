---
layout: post
title: "Ruby Tips - attr_accessor vs attr_writer vs attr_reader"
date: 2019-02-04 17:40:29 +0800
comments: true
categories: ruby ruby_tips
---

<!-- more -->

快速在 Ruby 的類別裡產生一對 getter 以及 setter 方法

* `attr_accessor`: `attr_reader` + `attr_writer`
* `attr_reader`: `def name; end`
* `attr_writer`: `def name=(new_name); end`

```ruby
class Animal
  attr_accessor :name

  def initialize
    @name = 'Dog'
  end
end


animal = Animal.new

animal.name
# => "Dog"
animal.name = 'Cat'
animal.name
# => "Cat"
```

相當於

```ruby
class Animal
  def initialize
    @name = 'Dog'
  end
  
  # attr_reader
  def name
  	 @name
  end
  
  # attr_writer
  def name=(new_name)
  	 @name = new_name
  end
end
```

參考文件

[Ruby 語法放大鏡之「attr_accessor 是幹嘛的?」](https://kaochenlong.com/2015/03/21/attr_accessor/)
