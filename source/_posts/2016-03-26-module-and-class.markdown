---
layout: post
title: "Ruby - Module vs Class"
date: 2016-03-26 09:34:18 +0800
comments: true
categories: ruby
---

在 ruby 中經常會使用到 `Class`，也常常看到 `Module`
那到底什麼時候要用哪個?

<!-- more -->

先看看兩者的差異

# Module

* 無法 instantiated（也就是沒有 new method）
* 可以被 include & extend

# Class

* 可以 instantiated
* 無法被 include

簡單的來說，module 有點像是沒有 new 的 Class

並且可以用 `include` & `extend` mixin 到 Class 裡面，變成 Class 的 `class method` or `instance method`

[Ruby 中的 Include Extend Require Load](http://mgleon08.github.io/blog/2016/02/24/include-extend-require/)

基本上兩種方式都有辦法達成同樣的效果，但差別就在於維護上

# 維護

用 Class 的話，勢必一定要先 new 出來，並且預設會執行 `initialize` 這個method
就可以預先設定好一些參數，看起來就會比較簡潔

```ruby
class Pepole
  def initialize(name)
    @name = name
  end

  def hi
    "hi, #{@name}"
  end

  def yo
    "yo, #{@name}"
  end
end

p = People.new('leon')
p.hi #=> 'hi leon'
p.yo #=> 'yo leon'
```

```ruby
class Test
  class << self
    def hello
      "hello"
    end
  end

  def here
    "here"
  end
end

puts Test.hello
puts Test.new.here
```

Module

```ruby
module Man
  module_function
  #也可以在每個 method 前面加上 self
  #也能改成 extend self，但比較偏好 module_function

  def hi(name)
    "hi, #{name}"
  end

  def yo
    "yo, #{name}"
  end
end

Man.hi('leon') #=> 'hi leon'
Man.yo #=> 'yo Man' name 應該是 keyword 顯示 module 名稱
```

另外是當 Class include 很多 module 時，都會變成 Class 的 `class method` or `instance method`，這時就很難去分辨是從哪個 module 來的 method。

因此使用上，看習慣，可維護性等等，去判斷要用 module or class
再細分要放在 `cocern` or `service object` or `lib`

官方文件：

* [module_function](http://apidock.com/ruby/Module/module_function)

參考文件：

* [module include extend](http://mgleon08.github.io/blog/2016/02/24/include-extend-require/)
* [class與module的差異? class如何繼承](http://railsfun.tw/t/class-module-class/402)
* [Difference between a class and a module](http://stackoverflow.com/questions/151505/difference-between-a-class-and-a-module)
* [Rails利用Module整理Model
](http://motion-express.com/blog/20141011-rails-module-model)
[Ruby class 基本的覆寫(override)及繼承(inheritance)](http://motion-express.com/blog/20141209-ruby-class-inheritance)
[Ruby當中的class method和instance method差在哪？](http://motion-express.com/blog/20141208-class-method-and-instance-method)
