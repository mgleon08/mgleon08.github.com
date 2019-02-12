---
layout: post
title: "Ruby - super vs super()"
date: 2019-02-04 18:19:10 +0800
comments: true
categories: ruby
---

<!-- more -->

`super` 可以在同一個 method，呼叫上層的同個 method，但有沒有 `()` 行為上會有點不一樣

# super

return `ArgumentError` 代表 `super` 會將 `Dog say` 的參數，帶到 `Animal say`，因此造成 `ArgumentError`

如果剛好 `Animal say` 也有帶參數，那就不會 error

```ruby
class Animal
  def say
    puts 'hi'
  end
end

class Dog < Animal
  def say(text)
    super

    puts text
  end
end

Dog.new.say('Woo')
# => ArgumentError (wrong number of arguments (given 1, expected 0))
```

# super()

而 `super()` 代表不帶任何參數的呼叫 `Animal say`

```ruby
class Animal
  def say
    'hi'
  end
end

class Dog < Animal
  def say(text)
    super()

    text
  end
end

Dog.new.say('Woo')
```

參考文件

* [Super keyword in Ruby](https://stackoverflow.com/questions/4632224/super-keyword-in-ruby)

