---
layout: post
title: "Nothing is Something by Sandi Metz"
date: 2018-07-26 21:36:02 +0800
comments: true
categories: rails
---

這是 Sandi Metz 2015 的演講，雖然有點舊，但還是很不錯，來紀錄一下

<!-- more -->

### Smalltalk Infected

一開始先介紹 `if` 可以改寫為自己的 method

```ruby
true.class # TrueClass
false.class # FalseClass
```

```ruby
class TrueClass
 def if_true
    yield
    self
 end
 
 def if_false
    self
 end
end

class FalseClass
 def if_true
    self
 end
 
 def if_false
    yield
    self
 end
end
```

```ruby
(1 == 1).if_true{ puts "evaluated block" }
evaluated block
# => true
```

接下來改寫為 `Object` `FalseClass` `NilClass` 因為在 ruby 的世界中，除了 `false` 和 `nil` 是 "Falsy"，其他都是 "truthy"

```ruby
class Object
 def if_true
    yield
    self
 end
 
 def if_false
    self
 end
end

class FalseClass
 def if_true
    self
 end
 
 def if_false
    yield
    self
 end
end

class NilClass
 def if_true
    self
 end
 
 def NilClass
    yield
    self
 end
end
```

```ruby
(1==1).if_true{ puts 'a' }.if_false{ puts'b' }
a
# true
```

但我們並不想改寫 ruby 原本就有的 method，而是將上面的技巧應用在需要的地方

### Condition Averse

Sometimes nil is nothing

```ruby
ids = ['pig', '', 'sheep']
animals = ids.map {|id| Animal.find(id)}
# => [#<Animal:0x007f94b290ae90 @name="pig">, nil,
     #<Animal:0x007f94b290ae18 @name="sheep">]
     
animals.each { |animal| puts animal.name }
# => 'pig'
# NoMethodError: undefined method `name' for nil:NilClass

animals.each { |animal| puts animal.nil? ? 'no animal' : animal.name }
# => 'pig'
#    'no animal'
#    'sheep'

animals.each { |animal| puts animal && animal.name }
animals.each { |animal| puts animal.try(:name) }
animals.each { |animal| puts animal.nil? ? '' : animal.name }
animals.each { |animal| puts animal == nil ? '' : animal.name }
animals.each { |animal| puts animal.is_a?(NilClass) ? '' : animal.name }
# => 'pig'
#    empty string
#    'sheep'
```

### Message Centric

新增 `MissingAnimal` class

```ruby
class Animal
  def name
    ...
  end 
end

class MissingAnimal 
  def name
    'no animal'
  end 
end

ids = ['pig', '', 'sheep']
animals = ids.map {|id| Animal.find(id) || MissingAnimal.new}
# => [#<Animal: @name="pig">, #<MissingAnimal:>, #<Animal: @name="sheep">]

animals.each { |animal| puts animal.name } 
# => 'pig'
#    'no animal'
#    'sheep'
```

但是這樣反而對 `MissingAnimal` 會有 dependency，接著在外面再包一層，將 dependency 封裝起來

```ruby
class GuaranteedAnimal 
  def self.find(id)
    Animal.find(id) || MissingAnimal.new 
  end
end

animals = ids.map { |id|GuaranteedAnimal.find(id) }
# => [#<Animal: @name="pig">, 
      #<MissingAnimal:>,
      #<Animal: @name="sheep">]
      
animals.each {|animal| puts animal.name }
# => 'pig'
#    'no animal'
#    'sheep'
```

### Abstraction Seeking

```ruby
class House
  def recite
    (1..data.length).map { |i| line(i) }.join("\n")
  end

  def line(number)
    "This is #{phrase(number)}.\n"
  end

  def phrase(number) 
    parts(number).join(" ")
  end
  
  def parts(number) 
    data.last(number)
  end

  def data
    [ 'the horse and the hound and the horn that belonged to',
    # ...
    'the malt that lay in',
    'the house that Jack built']
  end
end
```

接著 Implement `RandomHouse` `EchoHouse` without 'if' statements

用繼承 Inheritance?

```ruby
class RandomHouse < House 
  def data
    @data ||= super.shuffle 
  end
end

class EchoHouse < House 
  def parts(number)
    super.zip(super).flatten 
  end
end
```

但這樣一個要改寫 `data` 另一個改寫 `parts`，當有新需求 `RandomEchoHouse`，那不就要這兩個 method 在寫一次，也不可能只繼承其中一個

> Inheritance is for specialization is not for sharing code

改用組合 Composition 的方式來處理

```ruby
class House 
  attr_reader :formatter, :data
    
  def initialize(orderer: DefaultOrder.new, formatter: DefaultFormatter.new) 
    @formatter = formatter
    @data = orderer.order(DATA)
  end
  
  def parts(number) 
    formatter.format(data.last(number))
  end
  # ...
end

class DefaultOrder 
  def order(data)
    data
  end 
end

class RandomOrder 
  def order(data)
    data.shuffle 
  end
end

class DefaultFormatter 
  def format(parts)
    parts
  end 
end

class EchoFormatter 
  def format(parts)
    parts.zip(parts).flatten 
  end
end


House.new(orderer: RandomOrder.new).line(12)
House.new(formatter: EchoFormatter.new).line(12)
```

參考文件

* [[Video]RailsConf 2015 - Nothing is Something](https://www.youtube.com/watch?v=OMPfEXIlTVE)
* [[Silde] RailsConf 2015 - Nothing is Something](https://speakerdeck.com/skmetz/nothing-is-something-railsconf)
