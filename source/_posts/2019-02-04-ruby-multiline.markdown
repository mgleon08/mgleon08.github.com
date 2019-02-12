---
layout: post
title: "Ruby - multiline"
date: 2019-02-04 17:43:18 +0800
comments: true
categories: ruby
---

<!-- more -->

多行文字的輸入

* 第一個 `<<EOF`：表示把內容當作標準輸入程式stdin (Standard Input)
* 第二個 `EOF`：表示定義的 "文字流"（stream）終止
* `EOF` 要換什麼字都可以
* 可以加上 `*` 重複字串 `<<USAGE * 3`

### <<

* 最後面的 `USAGE`，前面不能有空格
* output 不會自行縮排

```ruby
def c
  <<USAGE
    1
    2
USAGE
end

c
# => "    1\n    2\n"
```

### <<-

* output 不會自行縮排

```ruby
def b
  <<-USAGE
    1
    2
  USAGE
end

b
# => "    1\n    2\n"
```

### <<~

* output 會自行縮排

```ruby
def a
  <<~USAGE
    1
    2
  USAGE
end

a
# => "1\n2\n"
```

參考文件

* [Multiline strings in Ruby 2.3 - the squiggly heredoc](https://infinum.co/the-capsized-eight/multiline-strings-ruby-2-3-0-the-squiggly-heredoc)
* [Ruby 2.3 new feature 之一: 多行字串更優美的寫法](https://ruby-china.org/topics/28501)
* [Ruby Ruby 多行字串 heredoc 詳解](https://ruby-china.org/topics/25983)
