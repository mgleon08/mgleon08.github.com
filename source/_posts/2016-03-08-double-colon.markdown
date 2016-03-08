---
layout: post
title: "double colon(::) 是啥? 能吃嗎？"
date: 2016-03-08 22:42:03 +0800
comments: true
categories: ruby
---

在 ruby 中常常會看到各種符號， 像是 `::` 在 ruby 就像是 `namespace` 的感覺。  
`::` 加在最前面，就是會去參照最上面的 `namespace`。

<!-- more -->

```ruby
MR_COUNT = 0        # constant defined on main Object class
module Foo
  MR_COUNT = 0
  ::MR_COUNT = 1    # set global count to 1
  MR_COUNT = 2      # set local count to 2
end
puts MR_COUNT       # this is the global constant
puts Foo::MR_COUNT  # this is the local "Foo" constant
```

```ruby
module Foo
  # We may not know about this in real big apps
  module Rails
    class Engine 
    end
  end

  class Engine1 < Rails::Engine
  end

  class Engine2 < ::Rails::Engine
  end
end

Foo::Engine1.superclass
 => Foo::Rails::Engine # not what we want

Foo::Engine2.superclass
 => Rails::Engine # correct
```

參考文件：  
[Ruby 語法放大鏡之「有時候會看到有兩個冒號寫法是什麼意思?」](http://blog.eddie.com.tw/2015/04/19/namespace/)  
[Double colons before class names in Ruby?](http://stackoverflow.com/questions/4819312/double-colons-before-class-names-in-ruby)  
[Ruby's double colon (::) operator usage differences](http://stackoverflow.com/questions/10482772/rubys-double-colon-operator-usage-differences)  
[Ruby : Ruby dot “.” and double Colon “::” Operators](https://cbabhusal.wordpress.com/2015/03/26/ruby-ruby-dot-and-double-colon-operators/)