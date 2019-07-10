---
layout: post
title: "Ruby - hash with default values"
date: 2019-02-04 18:12:51 +0800
comments: true
categories: ruby
---

<!-- more -->

`hash` 若沒給值，預設任何一個 key 都是 `nil`

```ruby
hash = {} # => {}
hash[:a]  # => nil
```

也可以給定 default value

```ruby
hash = Hash.new('hi') # => {}
hash[:a] # => "hi"

# or use default=
hash = Hash.new # => {}
hash.default = 'h1' # => "hi"
hash[:a] # => "hi"
```

但有時候會用到比較複雜的，`key -> array`

會發現如果用 `Hash.new([])` 實際上會回傳同樣的 `object_id`，因此每個 `key` 操作的 `object` 都會是同一個

```ruby
hash = Hash.new([]) # => {}
hash[:a] # => []
hash[:a] << 1 # => [1]
hash[:a] # => [1]

# 發現 a 的 1 也在
hash[:b] << 2 # => [1, 2]
hash[:b] # => [1, 2]

# 連 object_id 也是一樣
hash[:b].object_id == hash[:b].object_id # => true

# 最後 hash 其實都是空的
hash # => {}

# 必須用 <<= 相當於 += 才會存在 hash，但還是共用同一個 arry
hash[:c] <<= 3 # => [1, 2, 3]
hash # => {:c=>[1, 2, 3]}
hash[:d] = hash[:d] << 4 # => [1, 2, 3, 4]
hash # => {:c=>[1, 2, 3, 4], :d=>[1, 2, 3, 4]}
```

### why

那為什麼會這樣? 實際上是 `<<` 的問題，在之前的文章有提到 `<<` 每次都是回傳相同的 `object_id` 這就是造成的問題，因此可以改用 `+=` 每次都回傳不同的 `object_id`

```ruby
hash = Hash.new([]) # => {}
hash[:a] += [1] # => [1]
hash # => {:a=>[1]}
hash[:b] += [2] # => [2]
hash # => {:a=>[1], :b=>[2]}
```

### default_proc

另一個方式是使用 `block`，每次就會分配不同的 `object`

> [Hash new](http://ruby-doc.org/core-2.5.1/Hash.html#method-c-new) If a block is specified, it will be called with the hash object and the key, and should return the default value. It is the block's responsibility to store the value in the hash if required.

```ruby
hash = Hash.new { |h, k| h[k] = [] } # => {}
hash[:a].object_id # => 70228210560820
hash[:a] << 1 # => [1]
hash # => {:a=>[1]}
hash[:b] << 2 # => [2]
hash[:b].object_id # => 70228210545760
hash # => {:a=>[1], :b=>[2]}

# or
hash = Hash.new
hash.default_proc = Proc.new{ |h, k| h[k] = [] }
```

也可以透過遞迴 `default_proc` 建立一個動態深度的 hash

```ruby
hash = Hash.new { |hash, key| hash[key] = Hash.new(&hash.default_proc) }
# => {}

hash[:a][:b][:c][:d] = 'hi' # => "hi"
hash # => {:a=>{:b=>{:c=>{:d=>"hi"}}}}
```

# HashWithIndifferentAccess

最後是在 rails 中有這個 method 可以很方便地讓你不管是用 string 或是 symbol 都可以拿到值，在 params 中也是因為這個方法，因此兩種都取得到

```ruby
a = Hash.new
# => {}
a['hi'] = 123
# => 123
a['hi']
# => 123
a[:hi]
# => nil

b = HashWithIndifferentAccess.new
# => {}
b['hello'] = 321
# => 321
 b['hello']
#=> 321
b[:hello]
# => 321

aa = a.with_indifferent_access
# => {"hi"=>123}
aa['hi']
# => 123
aa[:hi]
# => 123
```

參考文件

* [Hash](https://ruby-doc.org/core-2.6/Hash.html)
* [Strange, unexpected behavior (disappearing/changing values) when using Hash default value, e.g. Hash.new([])](https://stackoverflow.com/questions/2698460/strange-unexpected-behavior-disappearing-changing-values-when-using-hash-defa)
* [Ruby Hashes and Default Values](https://keepthecodesimple.com/ruby-hashes-default-values/)
* [Ruby 語法放大鏡之「為什麼 Hash 好像有不同的寫法?」](https://kaochenlong.com/2016/04/23/different-hash-format/)
