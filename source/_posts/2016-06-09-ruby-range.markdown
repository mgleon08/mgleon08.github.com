---
layout: post
title: "Ruby Range"
date: 2016-06-09 20:13:54 +0800
comments: true
categories: ruby
---
經常會使用 range 去判斷某個值，是否在某個區間  
ruby 也提供很多好用的方法去判斷，cover?, include? 等等

<!-- more -->

#Range

```ruby
#... 不包含結尾
(-1..-5).to_a      #=> []
(-5..-1).to_a      #=> [-5, -4, -3, -2, -1]
('a'..'e').to_a    #=> ["a", "b", "c", "d", "e"]
('a'...'e').to_a   #=> ["a", "b", "c", "d"]

(0..2) == Range.new(0,2)    #=> true
(0..2) == (0...2)           #=> false
```

#begin/end first/last
```ruby
r1 = 3..6
r2 = 3...6
r1a, r1b = r1.first, r1.last    #=> 3, 6
r1c, r1d = r1.begin, r1.end     #=> 3, 6
r2a, r2b = r2.begin, r2.end     #=> 3, 6 (注意：不是3和5)
r1.first(2)                     #=> [3, 4]
```
#step
從 0..20 中取出 0，5，10，20

```ruby
a = 0..20
a.step(5).to_a
#=> [0, 5, 10, 15, 20]
```
#include?/cover?
判斷值，是否在 range 當中

```ruby
r = 1 .. 5
r.include?(1)     #=> true
r.include?(0)     #=> false
r.cover?(1)       #=> true
r.cover?(0)       #=> false

("a".."z").include?("ab")     # => false 
("a".."z").cover?("ab")       # => true 
```
主要差異是
  
* include? 會將所有值一一拿出來做比對，因此效率較差
* cover?   只會取出開頭和結尾，去比對，值 > 開頭 && 值 <= 結尾，效能比較好

官方文件：  
[ruby-doc Range](http://ruby-doc.org/core-1.9.3/Range.html)

參考文件：  
[What is the difference between Range#include? and Range#cover? ?](http://stackoverflow.com/questions/21608935/what-is-the-difference-between-rangeinclude-and-rangecover)    
[Apprentice Blog of the Week: Setting Date Ranges in Ruby](https://blog.8thlight.com/makis-otman/2014/09/03/setting-date-ranges-in-ruby.html)