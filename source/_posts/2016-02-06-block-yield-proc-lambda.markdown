---
layout: post
title: "Ruby 中的 Block & Yield & Proc & Lambda"
date: 2016-02-06 10:37:37 +0800
comments: true
categories: ruby
---

在學 ruby 時，經常會搞不清楚這四個，因為都非常相像!  

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
    self.each_with_index do |n, i|
      self[i] = yield(n)
    end
  end
end

array = [1, 2, 3, 4]

array.iterate! do |n|
  n ** 2
end

puts array.inspect

# => [1, 4, 9, 16]
```

###&

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
###Proc

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
###lambda

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


其實就是再把 `&block` 的寫法簡化

```ruby
def test
  yield
end

def test(&block)
  block.call
end
```

#procedure (proc)

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

#lambda 新寫法

```ruby
# bad
lambda = lambda { |a, b​​| a + b }
lambda.call(1, 2)

# good
lambda = ->(a, b) { a + b }
lambda.(1, 2)
```

參考文件：  
[理解Ruby的4种闭包：blocks, Procs, lambdas 和 Methods。](http://rubyer.me/blog/917/)  
[聊聊 Ruby 中的 block, proc 和 lambda](https://ruby-china.org/topics/10414)  
[程式區塊與 Proc](http://openhome.cc/Gossip/Ruby/Proc.html)  
[使用 lambda](http://openhome.cc/Gossip/Ruby/Lamdba.html)