---
layout: post
title: "Ruby Tips - Method Missing"
date: 2016-04-19 22:18:09 +0800
comments: true
categories: ruby ruby_tips
---

當 ruby 找不到 method 就會 call 這個 method

<!-- more -->

```ruby
class Post
  #自己建立 method_missing 呼叫
  def method_missing(method_name, *args)
    puts "You tried to call #{method_name} with these arguments: #{args}"
    super #default method_missing handling raises a NoMethodError
  end
end
Post.new.submit(1, "Here's a post.")

```

用找不到 method 會 call `method_missing` 的特性，直接自己定義 method_missing 去定義呼叫其他的 method

>它的執行效率並不好，所以只適合用在沒辦法預先知道方法名稱的情況下

```ruby
class Post
  DELEGATED_METHODS = [:username, :avatar]
  def initialize(user)
    @user = user
  end
  
  def method_missing(method_name, *args)
    if DELEGATED_METHODS.include?(method_name)
      @user.send(method_name, *args)
    else
      super #沒有在設定的 DELEGATED_METHODS 裡面，就呼叫 default method_missing handling raises a NoMethodError
    end
  end 
end
```

```ruby
class Post
  def initialize(text)
    @text = text
  end
  def to_s
    @text
  end
  def method_missing(method_name, *args)
    match = method_name.to_s.match(/^hash_(\w+)/) #找前面是 hash_
    if match
      @text << " #" + match[1]
    else
      super 
    end
  end
￼end


post = Post.new("HI")
post.hash_ruby
post.hash_metaprogramming
puts post
#=> HI #ruby #metaprogramming
```

###respond_to?

```ruby
post = Post.new 
post.respond_to?(:to_s) # => true
post.hash_ruby #再 method_missing 有定義所以呼叫得到
post.respond_to?(:hash_ruby) # => false #但在 respond_to 卻回傳 false
```

因此必須自己定義 respond_to?

```ruby
class Post
  def respond_to?(method_name)
    method_name =~ /^hash_\w+/ || super
  end 
end
```

但是 `post.method(:hash_ruby)` 還是會出現 `NameError: undefined method`

所以要改成另一個 `respond_to_missing?`

```ruby
class Post
  def respond_to_missing?(method_name) 
    method_name =~ /^hash_\w+/ ||super
  end 
end
```

# ￼DEFINE_METHOD REVISITED

```ruby
class Post
  def initialize(text)
    @text = text
  end
  def to_s
    @text
  end
  def method_missing(method_name, *args)
    match = method_name.to_s.match(/^hash_(\w+)/)
    if match #有 match 到 hash_ 就建立出新的 method
      self.class.class_eval do
        define_method(method_name) do
          @text << " #" + match[1]
        end
      end 
      send(method_name) #並且呼叫 method
    else
      super #沒有就 raises a NoMethodError
    end
  end
end

#當 call post.hash_codeschool 就會定義出下面的 method 

def hash_codeschool
  @text << " #" + "codeschool"
end
```

#const_missing
另一個跟 method_missing 一樣的，主要是常數找不到時會 call  
[const_missing](http://apidock.com/ruby/Module/const_missing)

官方文件：  
[method_missing](http://apidock.com/ruby/BasicObject/method_missing)  
[MatchData](http://ruby-doc.org/core-2.2.0/MatchData.html)  
[Regexp](http://ruby-doc.org/core-2.1.1/Regexp.html)

參考文件：    
[如何設計出漂亮的 Ruby APIs](https://ihower.tw/blog/archives/4797)  
[method_missing，一個Ruby 程序員的夢中情人](https://ruby-china.org/topics/3434)  
[respond_to? vs. respond_to_missing?](http://stackoverflow.com/questions/13793060/respond-to-vs-respond-to-missing)
