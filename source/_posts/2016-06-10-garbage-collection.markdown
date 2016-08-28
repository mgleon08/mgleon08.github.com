---
layout: post
title: "Garbage Collection (GC)"
date: 2016-06-10 12:18:42 +0800
comments: true
categories: ruby
---
在 ruby 當中，經常會看到 : 的符號，代表 symbol  

<!-- more -->

跟一般 string 的差異在於

* 同樣的 string，會產生不同的 記憶體
* 同樣的 symbol，一樣的記憶體

```ruby
3.times do
  puts "foo".object_id
end
#=>70302331020060
#=>70302331019980
#=>70302331019920

3.times do
  puts :foo.object_id
end
#=>1091868
#=>1091868
#=>1091868

#將字串 freeze 起來，object_id 也會是一樣
3.times do
  puts "foo".freeze.object_id
end
#=>70172682147320
#=>70172682147320
#=>70172682147320
```

#Garbage Collection

* 在 ruby 2.2 之前，symbol 所佔用的記憶體沒辦法被自動回收，要釋放就必須重啟動程式，因此會造成 memory leak 的問題

* 但在 2.2 之後，Symbol GC(Garbage Collection) ，那些動態用 to_sym 或 intern 長出來的 Symbol 就可以跟一般物件一樣被回收了。

###ruby2.1

```ruby
# Ruby 2.1
before = Symbol.all_symbols.size
100_000.times do |i|
  "sym#{i}".to_sym
end
GC.start
after = Symbol.all_symbols.size
puts after - before
# => 100001
```
###ruby 2.2
```ruby
# Ruby 2.2
before = Symbol.all_symbols.size
100_000.times do |i|
  "sym#{i}".to_sym
end
GC.start
after = Symbol.all_symbols.size
puts after - before
# => 1
```

參考文件：  
[Ruby 語法放大鏡之「有的變數變前面有一個冒號(例如 :name)，是什麼意思?」](http://kaochenlong.com/2016/04/25/string-and-symbol/)  
[Symbol GC in Ruby 2.2](https://www.sitepoint.com/symbol-gc-ruby-2-2/)  
[[译] 在 Ruby 2.2 中的 Symbol GC](http://grantcss.com/blog/2015/01/26/symbol-gc-in-ruby-2-dot-2/)  
[Ruby 2.2 的 可回收 symbol](https://ruby-china.org/topics/21498)  
[升级 Ruby 2.1 以及 GC 调整](https://ruby-china.org/topics/17575)  
[Ruby 的 GC 不释放内存给回系统的？](https://ruby-china.org/topics/13740)  
[画说 Ruby 与 Python 垃圾回收](https://ruby-china.org/topics/28127)