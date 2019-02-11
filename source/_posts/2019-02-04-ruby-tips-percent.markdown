---
layout: post
title: "Ruby - %Q, %q, %W, %w, %r, %s, %I, %i"
date: 2019-02-04 17:51:40 +0800
comments: true
categories: ruby
---

<!-- more -->

各種 ruby 的特殊符號用法

```ruby
# 處理跳脫字元、特殊符號如#、單引號、雙引號
%Q(Now is #{Time.now}) # => "Now is 2019-02-01 23:01:22 +0800"

# 處理跳脫字元、特殊符號如#、單引號、雙引號，但無法轉換變數
%q(Now is #{Time.now}) # => "Now is \#{Time.now}"

# 轉換成 array
%W(#{Time.now} hello say\ hi) # => ["2019-02-01 23:01:22 +0800", "#hello", "say hi"]

# 轉換成 array，但無法轉換變數
%w(#{Time.now} hello say\ hi) # => ["\#{Time.now}", "#hello", "say hi"]

# 執行 shell 指令，相當於 `$path`
%x($path) # => ""

# 轉正規表達式 regular expressions
%r(/home/) # => /\/home\//

# 轉 string
%s(foo bar) # => :"foo bar"
%s(foo
bar) # => :"foo \nbar"

# 轉 symbol array
%I(a b c) # => [:a, :b, :c]

# 轉 symbol array，但無法轉換變數
%i(a b c) # => [:a, :b, :c]
```

參考文件

* [%Q, %q, %W, %w, %x, %r, %s](https://simpleror.wordpress.com/2009/03/15/q-q-w-w-x-r-s/)
