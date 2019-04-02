---
layout: post
title: "Ruby - case when"
date: 2019-02-04 17:38:58 +0800
comments: true
categories: ruby
---

<!-- more -->

# Basic

```ruby
num = 8

case num
  when 1, 3, 5, 7, 9
    puts "#{num} is odd"
  when 2, 4, 6, 8, 10
    puts "#{num} is even"
end

# 8 is even
```


```ruby
num = 8

case num
  when 1..5
    puts "hi"
  when 6..10
    puts "hello"
end

# hello
```

```ruby
case num
  when 1..5 then puts "hi"
  when 6..10 then puts "hello"
end
```

# case when, checking the class type

```ruby
a = "Zigor"
case a
when String
  puts "Its a string"
when Fixnum
  puts "Its a number"
end

# Its a string
```

# case when and regular expressions

```ruby
string = "I Love Ruby"
# string = "I Love Python"

case string
when /Ruby/
  puts "string contains Ruby"
else
  puts "string does not contain Ruby"
end

# string contains Ruby
```

# case when and lambdas


```ruby
num = 76

case num
when -> (n) { n % 2 == 0 }
  puts "#{num} is even"
else
  puts "#{num} is odd"
end

# 76 is even
```
