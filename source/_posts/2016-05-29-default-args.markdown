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

在處理應該存在的哈希鍵時，使用 fetch。

```ruby
heroes = { batman: 'Bruce Wayne', superman: 'Clark Kent' }
# bad - if we make a mistake we might not spot it right away
heroes[:batman] # => "Bruce Wayne"
heroes[:supermann] # => nil

# good - fetch raises a KeyError making the problem obvious
heroes.fetch(:supermann)
```

在使用 fetch 時，使用第二個參數設置默認值而不是使用自定義的邏輯。

```ruby
batman = { name: 'Bruce Wayne', is_evil: false }

# bad - if we just use || operator with falsy value we won't get the expected result
batman[:is_evil] || true # => true

# good - fetch work correctly with falsy values
batman.fetch(:is_evil, true) # => false
```

盡量用 fetch 加區塊而不是直接設定默認值。

```ruby
batman = { name: 'Bruce Wayne' }

# bad - if we use the default value, we eager evaluate it
# so it can slow the program down if done multiple times
batman.fetch(:powers, get_batman_powers) # get_batman_powers is an expensive call

# good - blocks are lazy evaluated, so only triggered in case of KeyError exception
batman.fetch(:powers) { get_batman_powers }
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