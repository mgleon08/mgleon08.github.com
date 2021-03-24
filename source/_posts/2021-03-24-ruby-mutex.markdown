---
layout: post
title: "Ruby - Mutex"
date: 2021-03-24 23:06:27 +0800
comments: true
categories: ruby
---

<!-- more -->

有時候會希望一個 block 東西都執行完，在執行下一個，如果是 db 的話可以用 [樂觀鎖 與 悲觀鎖 Optimistic Locking & Pessimistic Locking](https://mgleon08.github.io/blog/2017/11/01/optimistic-locking-and-pessimistic-locking/)，避免搶資源問題

但如果是像要發兩個 request 就可以使用 `Mutex` ，但要小心不要 deadlock (同時使用兩個，A 等待 B, B 等待 A)，可以搭配 `ConditionVariable`，讓它等待另一個 signal 之後再繼續執行

> Mutex implements a simple semaphore that can be used to coordinate access to shared data from multiple concurrent threads.

```ruby
# 必須一開始就宣告，如果直接用  Mutex.new.synchronize 會導致每個用的 lock 不一樣
lock = Mutex.new

100.times do |num|
  Thread.new do
    lock.synchronize do
      puts "1. #{num}"
      sleep 3
      puts "2. #{num}"
      sleep 3
    end
  end
end
```

# Reference

- [https://ruby-doc.org/core-2.6.3/Mutex.html](https://ruby-doc.org/core-2.6/Mutex.html)
- [https://ruby-doc.org/core-2.6.3/ConditionVariable.html](https://ruby-doc.org/core-2.6.3/ConditionVariable.html)
- [Working with Multithreaded Ruby Part I](https://dev.to/enether/working-with-multithreaded-ruby-part-i-cj3)
- [Working With Multithreaded Ruby Part II](https://dev.to/enether/working-with-multithreaded-ruby-part-ii-5e3)
