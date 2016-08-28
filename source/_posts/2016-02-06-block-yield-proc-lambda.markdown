---
layout: post
title: "Ruby 中的 Block & Yield & Proc & Lambda & method"
date: 2016-02-06 10:37:37 +0800
comments: true
categories: ruby
---

在學 ruby 時，經常會搞不清楚這幾個，因為都非常相像!  

<!-- more -->

#block

什麼是 block ?

在 ruby 中 `block` 就是只 `do..end` || `{}` 包起來的區塊，就稱為 `block`。

```ruby
[1,2,3].each do |i|
	puts i
end
# => 1,2,3

[1,2,3].each {|i| puts i}

# => 1,2,3
```

基本上兩個是等價的，但是慣例上，一行會用 `{}` ，多行則是用 `do..end`  
`block` 無法單獨存在，必須放在 `method` 後面

另外 `block` 是可以傳遞的，用 `&` 表示


#yield

`yield` 其實就是調用 `block` 的一種方式  

```ruby
class Array
  def iterate!
    self.each_with_index do |n, i| #[1, 2, 3, 4].each_with_index
      self[i] = yield(n)           #等於於 self[i] = n ** 2 
    end
  end
end

array = [1, 2, 3, 4]

# array 呼叫 iterate! method 並將後面 do 帶入
array.iterate! do |n|
  n ** 2
end

puts array.inspect

# => [1, 4, 9, 16]
```

###& 區塊引述傳遞 寫法

若要調用在變數，前綴一個『&』符號，並且要放在最後一個

```ruby
class Array
  def iterate!(&code)
    self.each_with_index do |n, i|
      self[i] = code.call(n)
    end
  end
end

array = [1, 2, 3, 4]

array.iterate! do |n|
  n ** 2
end

puts array.inspect
```

其實 ruby 自動會將 後面的 block 轉換成 proc

```ruby
a = Proc.new{|n| n ** 2}
array.iterate!(&a)

#這樣也成立，必須要加上 & 代表 block
```

###Proc 寫法

```ruby
class Array
  def iterate!(code)
    self.each_with_index do |n, i|
      self[i] = code.call(n)
    end
  end
end

array_1 = [1, 2, 3, 4]

square = Proc.new do |n|
  n ** 2
end

array.iterate!(square)

puts array.inspect
```

###yield 其實就是再把 `&block` 的寫法簡化

以下三種相等

```ruby
def test
  yield
end

def test(&block)
  block.call
end

def test
  Proc.new.call
end
```

###lambda 寫法

```ruby
class Array
  def iterate!(code)
    self.each_with_index do |n, i|
      self[i] = code.call(n)
    end
  end
end

array = [1, 2, 3, 4]

array.iterate!(lambda { |n| n ** 2 })

puts array.inspect
```

#&
& 實際上是會觸發物件的 `to_proc` 方法，並嘗試指定給 & 變數  
因此可以在物件上定義 `to_proc`，然後使用 & 來觸發

```ruby
class People
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def self.to_proc
    Proc.new { |people| people.name }
  end
end

a = People.new('adler')
b = People.new('kai')
c = People.new('eugene')

print [a, b, c].map(&People)
# ["adler", "kai", "eugene"]
```
symbol 都有 `to_proc` 這個 method 因此可以改寫成

```ruby
:upcase.to_proc.call("abc")
#=> "ABC"

:downcase.to_proc.call("ABC")
#=> "abc"
```

有些 method 甚至連 `&` 也可以省略

```ruby
[1, 2, 3].reduce { |sum, element| sum += element }
# => 6
[1, 2, 3].reduce(&:+)
# => 6
[1, 2, 3].reduce(:+)
# => 6
```

###proc into block

Calling a method with & in front of a parameter  

```ruby
tweets.each(&printer) 
```

turns a proc into block 

###block into a proc

Defining a method with & in front of a parameter  

```ruby
def each(&block)
```
turns a block into a proc,so it can be assigned to parameter

###togeter

```ruby
class Timeline
  attr_accessor :tweets
  def each(&block) #block into a proc
    tweets.each(&block) #proc into block
  end 
end
```

###加上 symlbol

```ruby
tweets.map { |tweet| tweet.user }
#上下相等
tweets.map(&:user)
```

###optional

```ruby
def print
  if block_given? #可判對是否有傳進 block
    tweets.each { |tweet| puts yield tweet }
  else
    puts tweets.join(", ")
  end
end
```

###closure

```ruby
def tweet_as(user)
  lambda { |tweet| puts "#{user}: #{tweet}" }
end
gregg_tweet = tweet_as("greggpollack")
#=> lambda { |tweet| puts "greggpollack: #{tweet}" }
gregg_tweet.call("Mind blowing!")
# => greggpollack: Mind blowing!
```

#procedure (proc)

proc 都會有 `.call` 的 method 來呼叫

```ruby
def who_am_i(&block)
  block.class
end

puts who_am_i {}

# => Proc
```

有此理解，透過 `&block` 將程式碼傳遞過去後，其實它就是 `proc`。

跟 `block` 不同的地方是，`proc` 是可保存的

```ruby
pro = Proc.new {|a|  puts a}
#=> <Proc:0x007fcb23ad2640@(irb):1>

pro = Proc.new do |a|
  puts a
end

pro.call(123)
123
#=>nil

pro.(123) #非正規用法
123
#=>nil

pro(123)
123
#=> 123
```

此時就會被 `pro` 存起來，因此引用時就不需加上 `&`

#lambda

lambda 與 method 用法相同，概念是一樣的  
不同的是 `Method` 是有名字的method，而 `lambda` 是匿名 method

```ruby
lam = lambda{|a| puts a}
#=> <Proc:0x007fcb23aa1860@(irb):1 (lambda)>

lam.class
#=> Proc 有此可知，lambda 的原型是 proc

lam = lambda do |a|
  puts a
end

lam(123)
#NoMethodError: undefined method `l' for main:Object

lam.(123) #非正規用法
123
#=> nil

lam.call(123)
123
#=> nil
```

lambda 跟 proc 非常類似，主要有兩個差異  

###1.lambda 會檢查參數的個數

```ruby
def args(code)
  one, two = 1, 2
  code.call(one, two)
end

args(Proc.new{|a, b, c| puts "Give me a #{a} and a #{b} and a #{c.class}"})
# => Give me a 1 and a 2 and a NilClass

args(lambda{|a, b, c| puts "Give me a #{a} and a #{b} and a #{c.class}"})
# *.rb:8: ArgumentError: wrong number of arguments (2 for 3) (ArgumentError)
```

###2.lambda 的return 會繼續執行，proc 則會直接終止

`lambda` 比較像是一個 method 的 return，會 return 值回去，並且繼續執行   
`proc` 則是比較像是 一整個 method 的 return，return 就結束

```ruby
def proc_return
  Proc.new { return "Proc.new"}.call
  return "proc_return method finished"
end

def lambda_return
  lambda { return "lambda" }.call
  return "lambda_return method finished"
end

puts proc_return
=>Proc.new
#=> nil

puts lambda_return
=>lambda_return method finished
#=> nil
```

###使用時機

`proc` 若作用域是在外面，無法將 return 傳進去，必須用 `lambda`  
在某些情況下，使用 `lambda` 會比 `proc` 還簡約。

```ruby
def generic_return(code)
  one, two    = 1, 2
  three, four = code.call(one, two)
  return "Give me a #{three} and a #{four}"
end

puts generic_return(lambda { |x, y| return x + 2, y + 2 })

puts generic_return(Proc.new { |x, y| return x + 2, y + 2 })

puts generic_return(Proc.new { |x, y| x + 2; y + 2 })

puts generic_return(Proc.new { |x, y| [x + 2, y + 2] })

# => Give me a 3 and a 4
# => *.rb:9: unexpected return (LocalJumpError)
# => Give me a 4 and a
# => Give me a 3 and a 4

# proc 需再用 array 包覆起來
```

#Ruby 1.9 lambda 新寫法

```ruby
# bad
lambda = lambda { |a, b​​| a + b }
lambda.call(1, 2)

# good
lambda = ->(a, b) { a + b }
lambda.(1, 2)
```

#method

```ruby
class People
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def coding(language)
    puts "like #{language}"
  end
end

a = People.new('leon')

a.method(:coding).class
#=> Method
a.method(:coding).call('ruby')
#=> like ruby
```

[取得 Method](http://openhome.cc/Gossip/Ruby/Method.html)

#Other

```ruby
even = ->(x) { (x % 2) == 0 }
even === 4 # => true
even === 9 # => false

#=== 等於是 call
```

```ruby
# wants a proc, a lambda, AND a block
def three_ways(proc, lambda, &block)
  proc.call
  lambda.call
  yield # like block.call
  puts "#{proc.inspect} #{lambda.inspect} #{block.inspect}"
end

anonymous = Proc.new { puts "I'm a Proc for sure." }
nameless  = lambda { puts "But what about me?" }

three_ways(anonymous, nameless) do
  puts "I'm a block, but could it be???"
end
 #=> I'm a Proc for sure.
 #=> But what about me?
 #=> I'm a block, but could it be???
 #=> #<Proc:0x00089d64> #<Proc:0x00089c74> #<Proc:0x00089b34>
```

#Style

用 `->(){}` 而不是 `lambda{}`

```ruby
# bad
l = lambda { |a, b| a + b }
l.call(1, 2)

# correct, but looks extremely awkward
l = ->(a, b) do
  tmp = a * 7
  tmp * b / 50
end

# good
l = ->(a, b) { a + b }
l.call(1, 2)

l = lambda do |a, b|
  tmp = a * 7
  tmp * b / 50
end
```

用 `proc` 而不是 `Proc.new`

```ruby
# bad
p = Proc.new { |n| puts n }

# good
p = proc { |n| puts n }
```

用 `proc.call()` 而不是 `proc[]` 或 `proc.()`

```ruby
# bad - looks similar to Enumeration access
l = ->(v) { puts v }
l[1]

# also bad - uncommon syntax
l = ->(v) { puts v }
l.(1)

# good
l = ->(v) { puts v }
l.call(1)
```

參考文件：  
[理解Ruby的4种闭包：blocks, Procs, lambdas 和 Methods。](http://rubyer.me/blog/917/)  
[聊聊 Ruby 中的 block, proc 和 lambda](https://ruby-china.org/topics/10414)  
[迭代器與程式區塊](http://openhome.cc/Gossip/Ruby/IteratorBlock.html)
[程式區塊與 Proc](http://openhome.cc/Gossip/Ruby/Proc.html)  
[使用 lambda](http://openhome.cc/Gossip/Ruby/Lamdba.html)  
[取得 Method](http://openhome.cc/Gossip/Ruby/Method.html)  
[Ruby的block, lambda, Proc與函式物件](http://chuyi.inow.tw/index.php/ruby%E7%9A%84block-lambda-proc%E8%88%87%E5%87%BD%E5%BC%8F%E7%89%A9%E4%BB%B6/)