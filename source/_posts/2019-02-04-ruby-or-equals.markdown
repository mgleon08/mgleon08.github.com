---
layout: post
title: "Ruby - ||= (or-equals) mean?"
date: 2019-02-04 17:28:16 +0800
comments: true
categories: ruby
---

<!-- more -->

`a ||= a` 等於 `a || a = b` 但不等於 `a = a || b`，意思上不一樣，但出來的結果會是一樣的

```ruby
# 以下相等
a || a = b
a ? a : a = b

if a then a
else a = b
end

# 以下相等
a = a || b
a = a ? a : b
if a then a = a
else a = b
end
```

```ruby
a, b = false, nil

a ||= b # => nil
a || a = b # => nil
a = a || b # => nil

a, b = nil, true

a ||= b # => true
a || a = b # => true
a = a || b # => true

a, b = 1, 2

a ||= b # => 1
a || a = b # => 1
a = a || b # => 1
```

參考文件

* [What does ||= (or-equals) mean in Ruby?](https://stackoverflow.com/questions/995593/what-does-or-equals-mean-in-ruby)
