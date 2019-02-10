---
layout: post
title: "Ruby Tips - == vs === vs eql? vs equal?"
date: 2019-02-04 17:25:33 +0800
comments: true
categories: ruby ruby_tips
---

<!-- more -->

* `==`: 比較兩邊的 `value`
* `eql?`: 比較兩邊的 `value` + `class type` 
* `equal?`: 比較兩邊的 `object_id`
* `===`: case equality ，比較像是 `A 描述了一個集合，B 屬於 A 嗎?`
 
```ruby
a, b = 1, 1
a == b     # => true
a === b    # => true
a.eql? b   # => true
a.equal? b # => true

a, b = 1, 1.0
a == b     # => true
a === b    # => true
a.eql? b   # => false
a.equal? b # => false

a, b = 1, "1"
a == b     # => false
a === b    # => false
a.eql? b   # => false
a.equal? b # => false
```

```ruby
Class  === Class  # => true
Object === Object # => true
Class  === Object # => true
Object === Class  # => true

# 就 Ruby Gotchas 解釋比較像是 Fixnum 有包含 1，1 卻不包含 Fixnum
1 === 1            # => true
Fixnum === 1       # => true
1 === Fixnum       # => false
Fixnum === Fixnum  # => false

(1..5) === 3           # => true
(1..5) === 6           # => false

Integer === 42          # => true
Integer === 'fourtytwo' # => false

/ell/ === 'Hello'     # => true
/ell/ === 'Foobar'    # => false

[1, 2, 3] === [1, 2, 3]
# => true
[1, 2, 3] === [1, 2]
# => false
```

參考文件

* [Ruby Gotchas](https://docs.google.com/presentation/d/1cqdp89_kolr4q1YAQaB-6i5GXip8MHyve8MvQ_1r6_s/edit#slide=id.g2fa7c811_0_12)
* [What does the “===” operator do in Ruby? [duplicate]](https://stackoverflow.com/questions/4467538/what-does-the-operator-do-in-ruby)
* [關於===你應該知道的幾件事兒 | Ruby](http://lazybios.com/2017/01/ruby-triple-equals-operator/)
