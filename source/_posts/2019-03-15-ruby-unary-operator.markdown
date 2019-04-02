---
layout: post
title: "Ruby - unary operator"
date: 2019-03-15 23:11:38 +0800
comments: true
categories: ruby
---

<!-- more -->

> In Ruby, a unary operator is an operator which only takes a single 'argument' in the form of a receiver.

`unary operator` likes `+` `-` `!` `~` `&` `*`..

```ruby
-8
# => -8
 
# 2.2.3 Ruby
-'test'
# NoMethodError: private method `-@' called for "test":String

# 2.4.1 之後似乎就不會有 error
-'test'
# => "test"
```

Add `-@` method to String class

```ruby
class String
  def -@
     self + " hello"
    end
end
# => :-@

-'test'
# => "test hello"
```

### Full Example

```ruby
class MagicString < String
  def +@
    upcase
  end

  def -@
    downcase
  end

  def ~
    # Do a ROT13 transformation - http://en.wikipedia.org/wiki/ROT13
    tr 'A-Za-z', 'N-ZA-Mn-za-m'
  end

  def to_proc
    Proc.new { self + " hello" }
  end

  def to_a
    [self.reverse]
  end

 def !
   swapcase
 end
end

str = MagicString.new("This is my string!")
p +str                   # => "THIS IS MY STRING!"
p ~str                   # => "Guvf vf zl fgevat!"
p +~str                  # => "GUVF VF ZL FGEVAT!"
p %w(a b).map(&str)      # => ["This is my string! hello", "This is my string! hello"]
p *str                   # => "!gnirts ym si sihT"

p !str                   # => "tHIS IS MY STRING!"
p (not str)              # => "tHIS IS MY STRING!"
p !(~str)                # => "gUVF VF ZL FGEVAT!"
```

Reference

* [Ruby’s Unary Operators and How to Redefine Their Functionality](http://www.rubyinside.com/rubys-unary-operators-and-how-to-redefine-their-functionality-5610.html)
