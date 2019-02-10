---
layout: post
title: "Ruby Tips - Class Macro (Ruby’s declarative style)"
date: 2016-06-09 20:14:09 +0800
comments: true
categories: ruby ruby_tips
---

Class Macro 就是在 rails 的 ActiveRecord 中，經常會看到，`validates` `belongs_to` `hsa_manay` 等等的宣告

<!-- more -->

```ruby
class User < ActiveRecord::Base
  validates  :name, presence: true
  belongs_to :group
  has_many   :posts
end
```

範例:

```ruby
module ActiveRecord
  class Base
    def self.has_many(name)
      puts "#{self} has many #{name}"
	
      #定義 Dynamic method
      define_method(name) do
        puts "Select * From #{name} Where.."
        puts "Return #{name}"
        []
      end
    end
  end
end

class Movie < ActiveRecord::Base
  #self.has_many(:reviews)
  has_many :reviews
  has_many :genres
end

class Project < ActiveRecord::Base
  has_many :tasks
end

movie = Movie.new
movie.reviews
project = Project.new
project.tasks

#=>Movie has many reviews
#=>Movie has many genres
#=>Project has many tasks
#=>Select * From reviews Where..
#=>Return reviews
#=>Select * From tasks Where..
#=>Return tasks
```

之前有寫過 
[Dynamic Classes & Methods](http://mgleon08.github.io/blog/2016/04/19/dynamic-classes-and-methods/)

另一個範例，可以動態的將取出來的值做改變

```ruby
module Precentage
  def transform(*columns, precentage: nil)
    columns.each do |column|
      #這裡做了 alias_method 主要是希望，如果想知道原本的價錢，還可以呼叫 xx_origin
      alias_method "#{column}_origin", column
      #可簡化為 alias "#{column}_origin", column
      if precentage
        #動態產生新的 method 去取代原本
        define_method(column) do
          send(precentage) * send("#{column}_origin")
        end
      end
    end
  end
end

class Book
  extend Precentage
  attr_reader :price, :precentage

  def initialize(options)
    @price      = options[:price]
    @precentage = options[:precentage]
  end

  transform :price, precentage: :precentage
end

book = Book.new(price: 100, precentage:0.8)
puts book.precentage
puts book.price
puts book.price_origin

#=> 0.8
#=> 80.0
#=> 100
```

參考文件：

* [How To Write "Macros" in Ruby](https://pragmaticstudio.com/blog/2015/4/14/ruby-macros)
