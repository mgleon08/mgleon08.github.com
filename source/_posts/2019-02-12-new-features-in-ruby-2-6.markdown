---
layout: post
title: "Ruby - New Features in Ruby 2.6"
date: 2019-02-12 21:13:17 +0800
comments: true
categories: ruby
---

ruby 2.6 多了一些有趣的新特性和功能，來了解一下

<!-- more -->

# Endless ranges

利用 `..` 代表無窮的範圍

> 早期的寫法 `100..Float::INFINITY`

```ruby
n = 123

case n
when 1..9 then 'Single digit'
when 10..99 then 'Two digit'
when 100.. then 'Three or more'
end

# => "Three or more"
```

但也要小心，會一直跑下去

```ruby
(1..).each {|index| puts index }  
```

# Kernel#then (Kernel#yield_self alias)

新增一個 `yield_self` 新的 alias `then`

主要跟 `tap` 相反，`tap` 最後是返回原本的 `object`，`then` 則是返回 `block`

大概是下面這樣的差別

```ruby
class Object
  def tap
    yield self
    self
  end
  
   def yield_self
    yield(self)
  end
end
```

範例

```ruby
3.next.then {|x| x**x }.to_s             
#=> "256"
"my string".yield_self {|s| s.upcase }   
#=> "MY STRING"
```

```ruby
cisbn = '978-1-93778-549-9'

cisbn.gsub('-', '')
  .then { |isbn| URI("#{API_URL}?q=isbn:#{isbn}") }
  .then { |uri| Net:HTTP.get(uri) }
  .then { |json_response| JSON.parse(json_response) }
  .then { |response| response.dig('items', 'volumeInfo') }
```

* [yield_self](https://ruby-doc.org/core-2.6.1/Object.html#method-i-yield_self)


# Enumerable#chain and Enumerator#+

```ruby
(1..3).chain((5..7), [9, 10]).to_a 
# => [1, 2, 3, 5, 6, 7, 9, 10]
(1..3).each + (5..7) + (9..10).to_a
# => [1, 2, 3, 5, 6, 7, 9, 10]
```

# Composition operators << and >> to Proc and Method

```ruby
f = proc{|x| x + 2}
g = proc{|x| x * 3}
(f << g).call(3) # -> 11; identical to f(g(3))
(f >> g).call(3) # -> 15; identical to g(f(3))
```

# Array#union and Array#difference


```ruby
a = [1, 2, 3]
b = [2, 3, 4]

a.union(b) # => [1, 2, 3, 4]
a.difference(b) # => [1]
a | b # => [1, 2, 3, 4]
a & b # => [2, 3]
a - b # => [1]
a && b # => [2, 3, 4]
a || b # => [1, 2, 3]
```

* [union](https://ruby-doc.org/core-2.6/Array.html#method-i-union)
* [difference](https://ruby-doc.org/core-2.6/Array.html#method-i-difference)


# Array#filter (Array#select alias)

主要是其他語言 `Javascript, PHP, Haskell, Java 8, Scala, R` 等等都是用 `fliter`

```ruby
[:foo, :bar].filter { |x| x == :foo }
```

# Enumerable#to_h with block

`to_h` 支援 block，就可以直接作轉換 `key`, `value`

```ruby
hash = { foo: 2, bar: 3 }
hash.to_h { |k, v| [k.upcase, v*v] } #=> { FOO: 4, BAR: 9 }

# ruby 2.5:
# hash.map { |k, v| [k.upcase, v*v] }.to_h
# hash.reduce({}) { |result, (k, v)| result.merge(k.upcase => v*v) }
```

# Enumerator::ArithmeticSequence

之前無法用 `first` 和 `last`

* `Range#step`
* `Numeric#step`

```ruby
(1..10).step(2).last
# 9

(1..10).step(2).last
# ruby 2.5:
# NoMethodError: undefined method `last' for #<Enumerator: 1..10:step(2)>
```

`%` is `step` alias

```ruby
((1..10) % 2).to_a
#  => [1, 3, 5, 7, 9]
```

另一個改變

```ruby
(1..10).step(2) == (1..10).step(2)
# false - Ruby 2.5 (and older)
(1..10).step(2) == (1..10).step(2)
# true - Ruby 2.6
```

# Merge multiple hashes

```ruby
a = { a: 1 }
b = { b: 2 }
c = { c: 3 }
a.merge(b, c)
# {:a=>1, :b=>2, :c=>3}


a.merge(b).merge(c)
# ruby 2.5:
# {:a=>1, :b=>2, :c=>3}
```

# Random.bytes

```ruby
Random.bytes(8)
# => "\xA4\xFB\xC4\x94\xC5U\xA0\x1A"

# ruby 2.5
Random.new.bytes(10)
# => "\xCEn@\xFA\x93\xB3\xB9\x80p\xA9"
```

# Range#=== now uses uses #cover? instead of #include?

`===`: `case equality`，原本是用 `include?` 的方式，改為用 `cover?`

* include? 會將所有值一一拿出來做比對，因此效率較差
* cover?   只會取出開頭和結尾，去比對，值 => 開頭 && 值 <= 結尾，效能比較好

> cover?
> 
> Returns true if obj is between the begin and end of the range.
> 
> This tests begin <= obj <= end when exclude_end? is false and begin <= obj < end when exclude_end? is true.

```ruby
("a".."z").cover?("c")  #=> true
("a".."z").cover?("5")  #=> false
("a".."z").cover?("cc") #=> true
(1..5).cover?(2..3)     #=> true
(1..5).cover?(0..6)     #=> false
(1..5).cover?(1...6)    #=> true

("a".."z").include?("g")   #=> true
("a".."z").include?("A")   #=> false
("a".."z").include?("cc")  #=> false
```
```ruby
require 'date'

case DateTime.now
when Date.today..Date.today+1
  puts 'matched'
else
  puts 'not matched'
end
```

* [cover?](https://ruby-doc.org/core-2.6.1/Range.html#method-i-cover-3F)
* [include?](https://ruby-doc.org/core-2.6.1/Range.html#method-i-include-3F)

參考文件

* [NEWS-2.6.0](https://github.com/ruby/ruby/blob/trunk/doc/NEWS-2.6.0)
* [What's new in Ruby 2.6](https://nithinbekal.com/posts/ruby-2-6/)
* [What’s new in Ruby 2.6?](https://medium.com/tailor-tech/whats-new-in-ruby-2-6-a4774f3631c1)
* [9 New Features in Ruby 2.6](https://www.rubyguides.com/2018/11/ruby-2-6-new-features/)
