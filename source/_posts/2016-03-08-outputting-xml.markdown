---
layout: post
title: "Outputting XML"
date: 2016-03-08 23:03:49 +0800
comments: true
categories: rails
---

在 `rails` 當中，可以用 `xml.builder` 來輕鬆的輸出 `xml`

<!-- more -->

#Format

可以直接在 `routes` ，設定 format 格式

```ruby
get  :l, :defaults => { :format => 'xml' }
```

或是 controller

```ruby
require 'csv'
class PeopleController < ApplicationController

def index
    @people = Person.all
    respond_to do |format|
      format.html
      format.json{ render :json => @person.to_json }
      format.xml { render :xml => @person.to_xml }
    end
end
```

#View

`index.xml.builder`

```ruby
xml.instruct!
xml.xml do
  xml.linkXml do
    xml.phone  @user
    xml.age    @age
  end
end

#<xml>
  #<linkXml>
    #<phone>1234-5678</phone>
    #<age>20</age>
  #</linkXml>
#</xml>
```

gem：  
[builder](https://github.com/jimweirich/builder)

參考文件：  
[Outputting XML Using Ruby on Rails
](https://richonrails.com/articles/outputting-xml-using-ruby-on-rails)
[Action View - 樣板設計](https://ihower.tw/rails4/actionview.html)