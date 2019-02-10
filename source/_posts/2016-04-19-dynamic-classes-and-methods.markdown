---
layout: post
title: "Ruby Tips - Dynamic Classes & Methods"
date: 2016-04-19 22:16:35 +0800
comments: true
categories: ruby ruby_tips
--- 

ruby 可以很方便的動態產生 Classes 和 Methods

<!-- more -->

#Struct

```ruby
class Post
  attr_accessor :user, :status
  def initialize(user, status)
    @user, @status = user, status
  end 
  
  def to_s
    "#{user}: #{status}"
  end
end

#上下相等

Post = Struct.new(:user, :status) do 
  def to_s
    "#{user}: #{status}"
  end
end
```

#send()

```ruby
post.say
= post.send(:say)
= post.send("say")

#也可以 call 到 private 的 method
```
#alias_method

```ruby
class Post
  attr_reader :foo #=> return @foo
  #一定要在定義好的 method 後面還 call 得到
  alias_method :bar, :foo #=> the same method 別名/原名

  def initialize(foo = [])
    @foo = foo
  end
end
```

###可改成 alias

```ruby
alias_method :bar, :foo
#上下相等
alias bar foo
```

#alias_attribute

```ruby
class Content < ActiveRecord::Base
  # has a title attribute
end

class Email < Content
  alias_attribute :subject, :title
end

e = Email.find(1)
e.title    # => "Superstars"
e.subject  # => "Superstars"
e.subject? # => true
e.subject = "Megastars"
e.title    # => "Megastars"
```

#define_method

```ruby

class Post
  def draft
    @status = :draft
  end
  
  def posted
    @status = :posted
  end
  
  def deleted
    @status = :deleted
  end
end

#上下相等

class Post
  states = [:draft, :posted, :deleted]
  states.each do |status|
    define_method status do
      @status = status
    end
  end 
end
```

#method()

```ruby

class Post
  def initialize(foo)
    @posts = posts
  end
  def contents
    @foo
  end
  def show_tweet(index)
    puts @foo[index]
  end 
end
```

```ruby

foo = ['Compiling!', 'Bundling...']
post = Post.new(foo)

content_method = post.method(:contents)
content_method.call
#=> ["Compiling!", "Bundling..."]

show_method = post.method(:show_tweet)

￼(0..1).each(&show_method)
￼#上下相等
show_method.call(0)
show_method.call(1)
```

#EX: log_method

```ruby

class MethodLogger
  def log_method(klass, method_name)
    klass.class_eval do
      alias_method "#{method_name}_original", method_name
      define_method method_name do |*args, &block|
        puts "#{Time.now}: Called #{method_name}"
        send "#{method_name}_original", *args, &block
      end
    end
  end
end

class Post
  def say_hi
    puts "Hi"
  end
end

logger = MethodLogger.new
logger.log_method(Post, :say_hi)
Post.new.say_hi

#=> 2016-04-05 12:14:01 +0800: Called say_hi
#=> Hi
```

#eval

可以機字串當成表達是去做處理

```ruby
def get_binding(str)
  return binding
end
str = "hello"
eval "str + ' Fred'"                      #=> "hello Fred"
eval "str + ' Fred'", get_binding("bye")  #=> "bye Fred"
```

官方文件：

* [Struct](http://ruby-doc.org/core-2.2.0/Struct.html)
* [send()](http://apidock.com/ruby/Object/__send__)
* [alias_method()](http://apidock.com/ruby/Module/alias_method)
* [method define_method](http://apidock.com/ruby/Module/define_method)
* [eval](http://apidock.com/ruby/Kernel/eval)


參考文件：

* [What does send() do in Ruby?](http://stackoverflow.com/questions/3337285/what-does-send-do-in-ruby)
* [alias vs alias_method](https://gist.github.com/plusor/6104625)
* [如何設計出漂亮的 Ruby APIs](https://ihower.tw/blog/archives/4797)
* [ruby动态编程](http://rainlife.iteye.com/blog/375531)
