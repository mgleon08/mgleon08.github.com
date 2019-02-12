---
layout: post
title: "Ruby - Regular Expressions Security"
date: 2019-02-04 18:10:36 +0800
comments: true
categories: regular ruby
---

<!-- more -->

之前在做 code review 發現的一個問題，平常沒注意到很容易忽略!

> `^` and `$` are the start and end of line anchors

```ruby
# url 的 regular expressions
/(^$)|(^(http|https):\/\/[a-z0-9]+([\-\.]{1}[a-z0-9]+)*\.[a-z]{2,5}(([0-9]{1,5})?\/?.*)?$)/ix
```

原因在於 `^` and `$` 會根據每一行去判斷，因此像以下

```ruby
javascript:exploit_code();/* # injection
http://hi.com # pass
*/
```

第二行會 pass，因此造成 injection 問題，改成以下

> `\A` and `\Z` are the start and end of string anchors

```ruby
\A(^$)|(^(http|https):\/\/[a-z0-9]+([\-\.]{1}[a-z0-9]+)*\.[a-z]{2,5}(([0-9]{1,5})?\/?.*)?$)\z
```

就會判斷一整串

參考文件

* [ruby regular-expressions security](https://guides.rubyonrails.org/security.html#regular-expressions)
* [Why do Ruby's regular expressions use \A and \z instead of ^ and $?](https://stackoverflow.com/questions/3632024/why-do-rubys-regular-expressions-use-a-and-z-instead-of-and)
