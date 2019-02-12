---
layout: post
title: "Ruby -  + vs concat vs <<"
date: 2019-02-04 17:14:36 +0800
comments: true
categories: ruby
---

<!-- more -->

* `+`: 每次都是不同的 `object_id`
* `concat`: 每次都是回傳相同的 `object_id`，因此速度上會比較快
* `<<`: 其實就是 concat 的 `alias`
* `interpolation`: 最後是字串的插值，`object_id` 也會不一樣，但是速度是裡面最快的([fast-ruby](https://github.com/JuanitoFatas/fast-ruby))，因為不需要再做處理

### string

```ruby
a, b, c, d, e, f = 'a', 'b', 'c', 'd', 'e', 'f'

a.object_id    # => 70178339985920
a = a + b      # => "ab"
a.object_id    # => 70178335597640

b.object_id    # => 70178339985900
b.concat(c)    # => "bc"
b.object_id    # => 70178339985900

c.object_id    # => 70178339985880
c << d         # => "cd"
c.object_id    # => 70178339985880

e.object_id    # => 70178339797960
e = "#{e}#{f}" # => "ef"
e.object_id    # => 70178339730520
```

### array

```ruby
a, b, c, d, e, f = ['a'], ['b'], ['c'], ['d'], ['e'], ['f']

a.object_id # => 70114372794940
a = a + b # => ["a", "b"]
a.object_id # => 70114372758680

c.object_id # => 70114372794840
c.concat(d) # => ["c", "d"]
c.object_id # => 70114372794840

e.object_id # => 70114372794740
e << f # => ["e", ["f"]]
e.object_id # => 70114372794740
```

參考文件

* [fast-ruby](https://github.com/JuanitoFatas/fast-ruby)
* [string concat](http://ruby-doc.org/core-2.4.0/String.html#method-i-concat)
