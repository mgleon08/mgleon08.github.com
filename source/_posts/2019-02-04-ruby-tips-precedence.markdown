---
layout: post
title: "Ruby - precedence with block( {} vs do..end ) and operator( && vs and, || vs or )"
date: 2019-02-04 17:45:07 +0800
comments: true
categories: ruby
---

<!-- more -->

按照 ruby 的慣例，單行用 `{}` 多行用 `do..end`，但實際上兩個有一些不一樣
就是`優先權(precedence)`

```ruby
puts [1,2,3].map{ |k| k+1 }
2
3
4
# => nil
puts [1,2,3].map do |k| k+1; end
#<Enumerator:0x0000010a06d140>
# => nil
```

因為 `{}` 的優先權比 `do..end` 高，因此括號的地方會如下

```ruby
puts ( [1,2,3].map{ |k| k+1 } )
puts ( [1,2,3].map ) do |k| k+1; end
```

另外一個 `優先權(precedence)` 的例子是

`&&` vs `and`，`||` vs `or`

可以發現 `&&` 和 `||` 優先權都大於 `and` 和 `or`，因此盡量使用 `&&` 和 `||`

```ruby
x = true && false  # => false
x # => false
x = true and false # 相當於 (x = true) and false
# => false
x # => true


y = false || true  # => true
y # => true
y = false or true  # 相當於 (y = false) and true
# => true
y # => false
```

參考文件

* [do..end vs curly braces for blocks in Ruby](https://stackoverflow.com/questions/5587264/do-end-vs-curly-braces-for-blocks-in-ruby)
* [Difference between “and” and && in Ruby?](https://stackoverflow.com/questions/1426826/difference-between-and-and-in-ruby)
