---
layout: post
title: "Ruby - array with default value"
date: 2019-02-04 18:10:41 +0800
comments: true
categories: ruby
---

<!-- more -->

`Array.new` 可以建立一個長度的 `default value`

這樣建立的話會對應到一樣的 `object`

```ruby
a = Array.new(2, Hash.new) # => [{}, {}]

a[0]['animal'] = 'dog' # => "dog"
a # => [{"animal"=>"dog"}, {"animal"=>"dog"}]

a[1]['animal'] = 'cat' # => "cat"
a # => [{"animal"=>"cat"}, {"animal"=>"cat"}]
```

必須給 `block` 每個 `object` 才會是獨立的

```ruby
a = Array.new(2) { Hash.new } # => [{}, {}]
a[0]['animal'] = 'dog' # => "dog"
a # => [{"animal"=>"dog"}, {}]
```

參考文件

* [Array new](https://ruby-doc.org/core-2.6.1/Array.html#method-c-new)


