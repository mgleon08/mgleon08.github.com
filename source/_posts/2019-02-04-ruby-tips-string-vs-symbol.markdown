---
layout: post
title: "Ruby - string vs symbol"
date: 2019-02-04 17:27:08 +0800
comments: true
categories: ruby
---

<!-- more -->

差別在於 Symbol 是 `immutable` 物件，每次的 object_id 都是一樣，String 則是 `mutable` 物件，每次都會建立新的 object_id

因此 symbol 會比較省記憶體且速度上會比較快

```ruby
:a.object_id
# => 716828
:a.object_id
# => 716828
:a.object_id
# => 716828
'a'.object_id
# => 70351233634000
'a'.object_id
# => 70351233616700
'a'.object_id
# => 70351233598880
```

* [Ruby 語法放大鏡之「有的變數變前面有一個冒號(例如 :name)，是什麼意思?」](https://kaochenlong.com/2016/04/25/string-and-symbol/)
