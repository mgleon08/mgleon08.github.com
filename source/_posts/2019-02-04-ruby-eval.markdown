---
layout: post
title: "Ruby - instance_eval vs class_eval vs module_eval"
date: 2019-02-04 17:54:57 +0800
comments: true
categories: ruby
---

<!-- more -->

# instance_eval

可以定義一個實例方法

> 官方解釋
>
> Evaluates a string containing Ruby source code, or the given block, within the context of the receiver (obj). In order to set the context, the variable self is set to obj while the code is executing, giving the code access to obj's instance variables and private methods


# class_eval

定義一個類別方法

> 官方解釋
>
>valuates the string or block in the context of mod, except that when a block is given, constant/class variable lookup is not affected. This can be used to add methods to a class. module_eval returns the result of evaluating its argument. The optional filename and lineno parameters set the text for error messages.

# module_eval

class_eval 的 alias

> class_eval is used for adding methods and attributes to an existing class.
>
> module_eval is used for adding methods and attributes to an existing modules.


```ruby
class KlassWithSecret
  def initialize
    @secret = 99
  end

  private
  def the_secret
    "Ssssh! The secret is #{@secret}."
  end
end
k = KlassWithSecret.new
k.instance_eval { @secret }           #=> 99
k.instance_eval { the_secret }        #=> "Ssssh! The secret is 99."
k.instance_eval { |obj| obj == self } #=> true

k.instance_eval do
  def get_secret
    @secret
  end
end

k.get_secret # => 99

# get_secret 只存在 k 的變數裡面
w = KlassWithSecret.new
w.get_secret # NoMethodError: undefined method `get_secret' for #<KlassWithSecret:0x007fa75508fc88 @secret=99>

# 那要如何才能共用呢? 必須用到 class_eval
KlassWithSecret.class_eval do
  def get_secret
    @secret # 類別的實體變數
  end
end

w.get_secret # => 99
```

參考文件

* [instance_eval](https://ruby-doc.org/core-2.5.1/BasicObject.html#method-i-instance_eval)
* [class_eval](http://ruby-doc.org/core-2.5.1/Module.html#method-i-class_eval)
* [Understanding class_eval and instance_eval](http://web.stanford.edu/~ouster/cgi-bin/cs142-winter15/classEval.php)
* [Ruby 基礎 理解 class_eval 和 instance_eval](https://ruby-china.org/topics/25739)
* [Ruby: class_eval vs module_eval](https://medium.com/rubycademy/ruby-class-eval-vs-module-eval-6c3cc24a070)
