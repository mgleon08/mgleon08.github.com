---
layout: post
title: ".nil? .empty? .blank? .present? 傻傻分不清楚？"
date: 2015-12-16 21:29:43 +0800
comments: true
categories: ruby rails api
---

在學習ruby on rails的過程中，常常會有一些非常神奇的語法
其中又有些語法非常的相像..所以常常會搞混

今天就來介紹一下這幾個很相似的 method

<!-- more -->

#.nil?
```ruby
nil.nil?       #true
[].nil?        #false
{}.nil?        #false
"".nil?        #false
" ".nil?       #false
"abc".nil?     #false
123.nil?       #false
```
* ruby method
* 任何物件都是false，只有nil是true

#.empty? & .any?
```ruby
nil.empty?     #NoMethodError: undefined method `empty?' for nil:NilClass
[].empty?      #true
{}.empty?      #true
"".empty?      #true
" ".empty?     #false
"abc".empty?   #false
123.empty?     #NoMethodError: undefined method `empty?' for 123:Fixnum
```

* ruby method
* 只要是空值就是 true（`空白` 不算空值）
* 相對的 method 是 `any?`

```ruby
nil.any?       #NoMethodError: undefined method `any?' for nil:NilClass
[].any?        #false
{}.any?        #false
"".any?        #NoMethodError: undefined method `any?' for "":String
" ".any?       #NoMethodError: undefined method `any?' for "":String
"abc".any?     #NoMethodError: undefined method `any?' for "":String
123.any?       #NoMethodError: undefined method `any?' for 123:Fixnum
```
但是要注意，String 並沒有提供 `.any?` 這個 method

#.blank? & .present?

```ruby
nil.blank?     #true
[].blank?      #true
{}.blank?      #true
"".blank?      #true
" ".blank?     #true
"abc".blank?   #false
123.blank?     #false
```

* Rails method
* 只要是 nil，空值都是 true
* 有點像是 `Object.nil? || Object.empty?` 的綜合體，但條件比 `.empty` 寬鬆一點，`空白` 也會是 true
* 相對的 method 是 `present?`

```ruby
nil.present?   #false
[].present?    #false
{}.present?    #false
"".present?    #false
" ".present?   #false
"abc".present? #true
123.present?   #true
```


>經過上面這樣看來，只有 `.blank?` 和 `.present?` 不會爆錯 (所以我都用這個(誤))
>主要還是要看當時的情境拉XD


#.persisted? & .new_record?

最後來介紹一下，如何判斷 Object 是否已經存在資料庫的 method。

```ruby
a = User.new   #還沒存入資料庫以前
a.persisted?   #false
a.new_record?  #true

a.save         #存入資料庫
a.persisted?   #true
a.new_record?  #false
```

#如何看物件有哪些方法？或是上一層是誰？

```ruby
Object.superclass       ＃查上一層是誰?
Object.ancestors        ＃查祖宗十八代是誰?
Object.methods          ＃查物件有哪些方法?
Object.respond_to? :new ＃查物件是否有這個方法?
```


官方文件：

* [.nil?](http://apidock.com/ruby/Object/nil%3F)
* [.empty?](http://apidock.com/rails/ActiveRecord/Associations/CollectionProxy/empty%3F)
* [.any?](http://apidock.com/ruby/Enumerable/any%3F)
* [.blank?](http://apidock.com/rails/Object/blank%3F)
* [.persisted?](http://apidock.com/rails/Object/present%3F)
* [.new_record?](http://apidock.com/rails/ActiveRecord/Base/new_record%3F)
* [.persisted?](http://apidock.com/rails/ActiveRecord/Persistence/persisted%3F)
* [.superclass](http://apidock.com/ruby/Class/superclass)
* [.ancestors](http://apidock.com/rails/ActiveRecord/Acts/Tree/InstanceMethods/ancestors)
* [.methods](http://apidock.com/ruby/Object/methods)
* [.respond_to?](http://apidock.com/ruby/Object/respond_to%3F)


