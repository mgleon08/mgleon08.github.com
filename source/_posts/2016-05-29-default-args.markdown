---
layout: post
title: "Default Args"
date: 2016-05-29 20:22:19 +0800
comments: true
categories: ruby
---

很常時候，需要給變數一個預設值，因此有以下方法，都可以設定預設值

<!-- more -->

#||

只要是空值，或是 false, nil 就會回傳後面的預設值

```ruby
h = { "a" => 100, "b" => false , 'c' => nil}
#=> {"a"=>100, "b"=>false, "c"=>nil}
h['a'] || 8
#=> 100
h['b'] || 8
#=> 8
h['c'] || 8
#=> 8
h['d'] || 8
#=> 8
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

#merge
只有在 merge 的參數裡有同樣的值，才會覆蓋掉 default 的值

```ruby
default = { "a" => 100, "b" => false , 'c' => nil }
args    = { "a" => 8 }

default.merge(args)
#=> {"a"=>8, "b"=>false, "c"=>nil}
```

官方文件：  
[fetch](http://apidock.com/ruby/Hash/fetch)  
[merge](http://apidock.com/ruby/Hash/merge)