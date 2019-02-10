---
layout: post
title: "Ruby Tips - A::B vs module A; class B"
date: 2019-02-04 18:18:05 +0800
comments: true
categories: ruby ruby_tips
---

<!-- more -->

在 ruby 中，`A::B` 相當於 `module A; class B` 但實際上會稍微有點不同的地方

# module A; class B

在 `module A` 和 `class B` 中間宣告的變數，會變成屬於 `A module` 的 scope，而 `class B` 也包在 `module A` 底下，因此可以訪問到 parent 的變數

1. `class B` 會 search 有沒有 `SCOPE`?
2. 沒有就往上層找 `module A` 有沒有 `SCOPE`?

# A::B

而在 `A::B` 裡面宣告是屬於 `A::B` 的 scope (也就是 `class B`)

1. `class A::B` search 有沒有 `SCOPE`?
2. 沒有往上層，就到了 global，並沒有 `module A` 這層

```ruby
SCOPE = 'global'

module A
  SCOPE = 'module A'
  class B
    def scope1
      SCOPE
    end
  end
end

class A::B
  def scope2
    SCOPE
  end
end

A::B.new.scope1 # => "module A"
A::B.new.scope2 # => "global"
```

```ruby
SCOPE = 'global'

module A
  class B
    def scope1
      SCOPE
    end
  end
end

class A::B
  SCOPE = 'A::B'
  def scope2
    SCOPE
  end
end

A::B.new.scope1 # => "A::B"
A::B.new.scope2 # => "A::B"
```

參考文件

* [Ruby 語法放大鏡之「有時候會看到有兩個冒號寫法是什麼意思?」](https://kaochenlong.com/2015/04/19/namespace/)
* [Ruby - Lexical scope vs Inheritance](https://stackoverflow.com/questions/15119724/ruby-lexical-scope-vs-inheritance)
