---
layout: post
title: "引數的傳遞 argument parameter"
date: 2016-03-26 09:28:59 +0800
comments: true
categories: ruby
---
經常會看到 `argument` 和 `parameter` 兩個很類似，卻代表的不同意義。  
另外也有許多符號 * ** & 可以使用。

<!-- more -->

#介紹
* argument(actual argument 實際的值) 引數  
「呼叫端」傳給呼叫對象的「值」

* parameter(formal parameter 代表參數的符號) = 參數  
「被呼叫端」用來接收引數的「變數」

#範例
有預設值的 `parameter` 要相連在一起

```ruby
def sample(a=1, b=2, c)
	puts [a, b, c]
end
```

傳入參數時，會先將值帶入沒有預設值的 `a`，在緊接著給最左邊的 `c`

```ruby
def sample(a, b=2, c=3)
  puts [a, b, c].inspect
end

sample(3)
#=> [3, 2, 3]

sample(3, 1)
#=> [3, 1, 3]
```

預設值的位置不同，結果也不一樣

```ruby
def sample(a=1, b=2, c)
  puts [a, b, c].inspect
end

sample(3)
#=> [1, 2, 3] #直接給沒有預設值的 c

sample(3, 1)
#=> [3, 2, 1] #有兩個值，會先將第一個值給有預設值的第一個 a，第二個才補到 c
```

#* Array
* 當 `parameter` 前面加上 `*` 代表 `Array` 的意思

```ruby
#必須和有預設值得放在一起
#若要放中間，最後面不能放預設值(因為有預設值要擺在一起)，可以放變數

def sample(a=1, b=2, *c)
  puts [a, b, c].inspect
end

sample(4,5,6,7,8)
#=> [4, 5, [6, 7, 8]]
```

```ruby
def sample(a, *b, c)
  puts [a, b, c].inspect
end

sample(1,2) #至少兩個參數
#=> [1, [], 2]

sample(1,2,3)
#=> [1, [2], 3]

sample(1,2,3,4)
#=> [1, [2, 3], 4]

sample(1,2,3,4,5)
#=> [1, [2, 3, 4], 5]
```

###另一種用法將 Array 展開來

```ruby
def sample(a=1, b=2, *c)
  puts [a, b, c].inspect
end

array = [4, 5, 6, 7, 8]
sample(array)
#=> [[4, 5, 6, 7, 8], 2, []] #將 numble 當成一個參數了ˊˋ

sample(*array)
#=> [4, 5, [6, 7, 8]] #將 Array 給展開來
```
```ruby
def sample(a=1, *b, c)
  puts [a, b, c].inspect
end

array = [4, 5, 6, 7, 8]

sample(array)
#=> [1, [], [4, 5, 6, 7, 8]]
sample(*array)
#=> [4, [5, 6, 7], 8]
```

#** Hash
* 當 `parameter` 前面加上 `**` 代表 `Hash` 的意思
* 一定要擺到最後一個
* 只能有一個
* 並且要傳入的是 `Hash`
* 傳入的值若不是在最後面，要加上 `{}`,預設會將後面所有的 `key-value` 包成一個 `Hash`

>即使後面不加 `**` 預設也會將後面的值都包成一個 hash

```ruby
def sample(a=1, b=2, **c)
  puts [a, b, c].inspect
end

sample('j':4, 'q':5, 'k':6)
#=> [1, 2, {:j=>4, :q=>5, :k=>6}]

sample({'j':4, 'q':5, 'k':6}, 3, 4) # 放在前面加上 {}
#=> [{:j=>4, :q=>5, :k=>6}, 3, 4]
```

```ruby
# 'b':1 傳入的值必須 key-value 並且要對到 key
def sample(a, b:2, c:3)  
  puts [a, b, c].inspect
end
sample('a':11, 'v':22, 'c':33, 's':44)
#=> [{:a=>11, :v=>22, :c=>33, :s=>44}, 2, 3]

sample({'a':11, 'v':22, 'c':33, 's':44}, 'b':100)
#=> [{:a=>11, :v=>22, :c=>33, :s=>44}, 100, 3]
```

```ruby
def sample(a, b, **c) # sample(a, b, c)
  puts [a, b, c].inspect
end

sample(4, 'a':1) 
#=> [4, {:a=>1}, {}]

sample(4, 5, 'a':1, 'b':2, 'c':3)
#=> [4, 5, {:a=>1, :b=>2, :c=>3}]

sample(4, 'a':1, 'v':2, 'c':4)
#=> [4, {:a=>1, :v=>2, :c=>4}, {}]

sample({'z':5}, {'a':1, 'v':2}, 'c':4)
#=> [{:z=>5}, {:a=>1, :v=>2}, {:c=>4}]

sample(4, {'a':1, 'v':2}, 'c':4)
#=> [4, {:a=>1, :v=>2}, {:c=>4}]
```

###另一種用法將 Hash 展開

```ruby
def sample(a, b, **c)
  puts [a, b, c].inspect
end

hash  = {:q=>4, :w=>5, :e=>6}
hash2 = {:r=>4, :t=>{:y=>5, :u=>6}}
hash3 = {:i=>4, :o=>5}

sample(1,hash)
#=>  [1, {:q=>4, :w=>5, :e=>6}, {}]

sample(1,hash, hash2)
#=> [1, {:q=>4, :w=>5, :e=>6}, {:r=>4, :t=>{:y=>5, :u=>6}}]

#將兩個 hash 合併成一個
sample(1 ,**hash, **hash2)
#=> [1, {:q=>4, :w=>5, :e=>6, :r=>4, :t=>{:y=>5, :u=>6}}, {}]

#展開後，後面就可以繼續塞 hash
sample(1,2,**hash, 'foo':123, 'bar':456)
#=> [1, 2, {:q=>4, :w=>5, :e=>6, :foo=>123, :bar=>456}]
```

```ruby
def sample(a:1, b:2, **c)
  puts [a, b, c].inspect
end

hash  = {:q=>4, :w=>5, :e=>6}
hash2 = {:r=>4, :t=>{:y=>5, :u=>6}}
hash3 = {:i=>4, :o=>5}

sample(**hash, **hash2, 'b':100)
#=> [1, 100, {:j=>4, :q=>5, :k=>6, :c=>{:q=>5, :k=>6}}]

sample(**hash, **hash2, 'test':100)
#=> [1, 100, {:q=>4, :w=>5, :e=>6, :r=>4, :t=>{:y=>5, :u=>6}}]

sample(**hash, **hash2, 'test':100, 'b':123)
#=> [1, 123, {:q=>4, :w=>5, :e=>6, :r=>4, :t=>{:y=>5, :u=>6}, :test=>100}]
```
#& 區塊傳遞

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

#綜合使用

```ruby
def sample(a, *b, **c, &d)  
  puts "a=#{a}, b=#{b}, c=#{c}"  
  d.call(100)  
end  
sample(1,2,3,x:4,y:5) { |x| puts x }  
  
#=> a=1, b=[2, 3], c={:x=>4, :y=>5}
#=> 100
```

若最後面是 * 不限制長度，塞很多變數，或是後面接 hash 都不會錯

```ruby
def sample(a, b, *c)
  puts [a, b, c].inspect
end

sample(1,2,3, 4, 5)
#=> [1, 2, [3, 4, 5]]

sample(1,2,'a':1, 'b':2)
#=> [1, 2, [{:a=>1, :b=>2}]]
```