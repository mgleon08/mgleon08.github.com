---
layout: post
title: "好用的 Hash method"
date: 2016-07-04 21:58:11 +0800
comments: true
categories: rails
---
在 rails 當中經常會使用 hash，rails 也提供很多方便的 methods

<!--more-->

#Hash

```ruby
Hash["a", 100, "b", 200]             #=> {"a"=>100, "b"=>200}
Hash[ [ ["a", 100], ["b", 200] ] ]   #=> {"a"=>100, "b"=>200}
Hash["a" => 100, "b" => 200]         #=> {"a"=>100, "b"=>200}
```
#merge & merge!

回傳新的 hash，後面一樣的 key 則會覆蓋前面的

>以後面的為優先

```ruby
h1 = { "a" => 100, "b" => 200 }
h2 = { "b" => 254, "c" => 300 }
h1.merge(h2)   #=> {"a"=>100, "b"=>254, "c"=>300}

#將重複的做其他處理
h1.merge(h2){|key, oldval, newval| newval - oldval}
#=> {"a"=>100, "b"=>54,  "c"=>300}
h1             #=> {"a"=>100, "b"=>200}
```

`merge!` 直接改變原先的 hash

```ruby
h1.merge!(h2)
#=> {"a"=>100, "b"=>254, "c"=>300}
h1
#=> {"a"=>100, "b"=>254, "c"=>300}
```

#reverse_merge & reverse_merge!

回傳新的 hash，前面一樣的 key 則會覆蓋後面的

通常用在指定hash的預設值

>以前面的為優先

```ruby
h1 = { "a" => 100, "b" => 200 }
h2 = { "b" => 254, "c" => 300 }

h1.merge(h2)
#=> {"a"=>100, "b"=>254, "c"=>300}
h1.reverse_merge(h2)
#=> {"b"=>200, "c"=>300, "a"=>100}
```

#deep_merge & deep_merge!

在兩個hash的鍵值相同，而值也是個hash的情況下

```ruby
h1 = {:a => {:b => 1}}
h2 = {:a => {:c => 2}}

h1.merge(h2)
#=> {:a=>{:c=>2}} error
h1.deep_merge(h2)
#=> {:a=>{:b=>1, :c=>2}}
```

#fetch

即使是 nil, false 也會回傳，只有在空值的時候回傳預設值

```ruby
h = { "a" => 100, "b" => false , 'c' => nil}
#=> {"a"=>100, "b"=>false, "c"=>nil}
h.fetch('a', 8)
#=> 100
h.fetch('b', 8)
#=> false
h.fetch('c', 8)
#=> nil
h.fetch('d', 8)
#=> 8
```

#except & except!

通常用在確保某些欄位不要被傳進來的參數修改到

```ruby
h1 = { a:100, b:200 }
h1.except(:a)
```
#symbolize_keys & symbolize_keys!

回傳新的 hash，key 值轉成 symbol

>通常用在確保 key 的一致性

```ruby
hash = { 'name' => 'Rob', 'age' => '28' }

hash.symbolize_keys
# => {:name=>"Rob", :age=>"28"}
```

#stringify_keys & stringify_keys!

回傳新的 hash，key 值轉成 string

alias `to_options` & `to_options!`

>通常用在確保 key 的一致性

```ruby
hash = { name: 'Rob', age: '28' }

hash.stringify_keys
#=> { "name" => "Rob", "age" => "28" }

#若有衝突已後面為優先
{"a" => 1, :a => 2}.stringify_keys
#=> {"a"=>2}
```

#slice & slice!

有 `!` 行為會不太一樣，要注意

```ruby
h1 = { "a" => 100, "b" => 200 }

h1.slice("a")
#=> {"a"=>100}
h1
#=> {"a"=>100, "b"=>200}

h1.slice!("a")
#=> {"b"=>200}
h1
#=> {"a"=>100}
```

#extract!
將需要的值提取出來，成為新的 hash

```ruby
h1 = { "a" => 100, "b" => 200 }
h1.extract!("a")
#=> {"a"=>100}
h1
#=> {"b"=>200}
```

#to_query

Alias for Hash#to_query

```ruby
{name: 'David', nationality: 'Danish'}.to_query
#=> "name=David&nationality=Danish"

{name: 'David', nationality: 'Danish'}.to_query('user')
# => "user%5Bname%5D=David&user%5Bnationality%5D=Danish"
```

#other

###attributes
將物件轉成 hash

```ruby
foo = Book.first
#=> #<Book id: 22, name: "book", desc: "desc", created_at: "xxx", updated_at: "xxx">

foo.attributes
#=> {"id"=>22, "name"=>"book", "desc"=>"desc", "star"=>1, "created_at"=>xxx, "updated_at"=>xxx}
```

官方文件：  
[Ruby Doc Hash](http://ruby-doc.org/core-2.1.5/Hash.html)  
[Hash apidock](http://apidock.com/rails/v4.2.1/Hash)  

參考文件：  
[ActiveSupport - 工具函式庫](https://ihower.tw/rails4/activesupport.html)  
