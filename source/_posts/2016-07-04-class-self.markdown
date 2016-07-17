---
layout: post
title: "class << self"
date: 2016-07-04 00:01:37 +0800
comments: true
categories: rails
---

有時會看到這種特別的寫法 `class << self`，有兩種不同的用法

<!-- more -->

#class method

將所以需要 `self.` 全部包在裡面，就不用一個一個 `self.`  
不過這比較看是個人偏好，有些人會覺得 `self.` 可讀性較高  

[class << self vs self.method with Ruby: what's better?](http://stackoverflow.com/questions/10964081/class-self-vs-self-method-with-ruby-whats-better)

```ruby
class Foo
	def self.bar
		puts 'bar'
	end
	def self.bar2
		puts 'bar2'
	end
end

puts Foo.bar
#bar
puts Foo.bar2
#bar2
```
上下相等

```ruby
class Foo
  class << self
    def bar
      puts 'bar'
    end
    def bar2
      puts 'bar2'
    end
  end
end

puts Foo.bar
#bar
puts Foo.bar2
#bar2
```

#對單一物件，宣告出一個類別

```ruby
class Foo
  def foo
    return 'foo'
  end
end

f1 = Foo.new
f2 = Foo.new

puts f1.foo # foo
puts f2.foo # foo

# 只對f1 修改方法
class << f1
  def foo
    return 'Edit foo'
  end
end

puts f1.foo # Edit foo
puts f2.foo # foo

# 只對f1 新增方法
class << f1
  def bar
    return 'bar'
  end
end

puts f1.bar # bar
puts f2.bar # undefined method `bar'
```

參考文件：  
[Ruby: class << self](http://wemee.blogspot.tw/2014/07/ruby-class.html)  
[class << self vs self.method with Ruby: what's better?](http://stackoverflow.com/questions/10964081/class-self-vs-self-method-with-ruby-whats-better)