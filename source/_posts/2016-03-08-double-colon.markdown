---
layout: post
title: "Ruby - double colon(::)"
date: 2016-03-08 22:42:03 +0800
comments: true
categories: ruby
---

在 ruby 中常常會看到各種符號， 像是 `::` 在 ruby 就像是 `namespace` 的感覺。
`::` 加在最前面，就是會去參照最上面的 `namespace`。

<!-- more -->

有點像是一個是絕對路徑，一個是相對路徑

```ruby
::A::B # like /A/B
# is an absolute path to the constant.

A::B # like ./A/B
# is a path relative to the current tree level.
```

```ruby
COUNT = 0        # constant defined on main Object class
module Foo
  COUNT = 0
  ::COUNT = 1    # set global COUNT to 1
  COUNT = 2      # set local COUNT to 2
end

# 因爲 constant 通常是不能變動的，但在 ruby 只會做緊告
# (irb):4: warning: already initialized constant COUNT
# (irb):1: warning: previous definition of COUNT was here
# (irb):5: warning: already initialized constant Foo::COUNT
# (irb):3: warning: previous definition of COUNT was here

puts COUNT       # this is the global constant
# => 1
puts Foo::COUNT  # this is the local "Foo" constant
# => 2
```

```ruby
module Rails
  class Engine
  end
end

module Foo
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
# => Foo::Rails::Engine
Foo::Engine2.superclass
# => Rails::Engine
```

參考文件：

* [Ruby 語法放大鏡之「有時候會看到有兩個冒號寫法是什麼意思?」](http://blog.eddie.com.tw/2015/04/19/namespace/)
* [Double colons before class names in Ruby?](http://stackoverflow.com/questions/4819312/double-colons-before-class-names-in-ruby)
* [Ruby's double colon (::) operator usage differences](http://stackoverflow.com/questions/10482772/rubys-double-colon-operator-usage-differences)
* [Ruby : Ruby dot “.” and double Colon “::” Operators](https://cbabhusal.wordpress.com/2015/03/26/ruby-ruby-dot-and-double-colon-operators/)
