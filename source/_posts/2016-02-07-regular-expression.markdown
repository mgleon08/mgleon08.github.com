---
layout: post
title: "正規表示式 Regular Expression"
date: 2016-02-07 15:40:31 +0800
comments: true
categories: regular
---

在網頁中經常要確認一些格式，像是身分證格式必須要 10 碼，開頭是大寫 A-Z ，第二個數字必須是 0 or 1，這時就能夠用 `Regular Expression` 來做判斷。

<!-- more -->

```ruby
身份證 
/[A-Z][12]\d{8}/

信箱
/\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i

不允許信箱中有多個點
/\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i
```

#範例

```ruby
regex = /([A-Z])([12])\d{8}/  #前面兩個()起來代表是有意義的，可以連同存取起來

match = "A123456789".match(regex)
#=> #<MatchData "A123456789" 1:"A" 2:"1">

match[0]
#=> "A123456789"
match[1]
#=> "A"
match[2]
#=> "1"
```

#Regex quick reference

```ruby
[abc]	#A single character of: a, b, or c
[^abc]	#Any single character except: a, b, or c
[a-z]	#Any single character in the range a-z
[a-zA-Z]	#Any single character in the range a-z or A-Z

^	#Start of line => /^ab/，開頭兩個有 ab 即可
$	#End of line => /ab$/，後面兩個有 ab 即可

\A	#Start of string
\z	#End of string
.	#Any single character

\s = [ \r\t\n\f] #Any whitespace character
\S = [^ \r\t\n\f] #Any non-whitespace character

\d = [0-9]	#Any digit
\D = [^0-9]  #Any non-digit

\w = [a-zA-Z0-9_] #Any word character (letter, number, underscore)
\W = [^a-zA-Z0-9_] #Any non-word character

\b	#Any word boundary
(...)	#Capture everything enclosed
(a|b)	#a or b

a?	#Zero or one of a
a*	#Zero or more of a
a+	#One or more of a

a{3}	#Exactly 3 of a
a{3,}	#3 or more of a
a{,6}	#ths most 6 of a
a{3,6}	#Between 3 and 6 of a

options: 
i #case insensitive 
m #make dot match newlines 
x #ignore whitespace in regex 
o #perform #{...} substitutions only once
```

參考文件：  
[正規表示式 Regular Expression](https://atedev.wordpress.com/2007/11/23/%E6%AD%A3%E8%A6%8F%E8%A1%A8%E7%A4%BA%E5%BC%8F-regular-expression/)

練習：  
[Rubular](http://rubular.com/)  
[Regex Cross­word](https://regexcrossword.com/)