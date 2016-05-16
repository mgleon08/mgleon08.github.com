---
layout: post
title: "自定 Module 和 class 檔案 Monkey Patching 手法"
date: 2016-02-01 22:19:13 +0800
comments: true
categories: rails
---

在 rails 當中，可以自訂一些好用方便的檔案，在適當的時機來使用。

<!-- more -->


#method
在 `lib/require/object.rb`，可以自行新增 class

or

`lib/core_ext/object/object.rb`  
`lib/core_ext/hash/hash.rb`

```ruby
class Object
  def is?(*objects)
    for object in objects
      return true if self == object
    end
    false
  end
end
```

#require
但要記得要在使用的檔案，先 require 才能夠使用

也可以直接在 `config/initializer` 新增 `require.rb` 檔案，將檔案 require 進來，就不用每個檔案上面都 require 了。

>`initializer` 裡面的檔案，程式一執行就會全部都執行了

require

`config/initializers/monkey_patches.rb`

```ruby
#單個檔案
require "#{Rails.root}/lib/require/object.rb"

#多個檔案
Dir["#{Rails.root}/lib/require/object.rb"].each do |file|
  require file
end

#lib 底下所有檔案
Dir["#{Rails.root}/lib/**/*.rb"].each do |file|
  require file
end

# 檔案 && 資料夾名稱可自訂 ex: monkey_patches
```

#使用

```ruby
"hello".is?("yes", "no", "hello)
#=> true

"hello".is?("yes", "no")
#=> false
```

#Refinement

「猴子補丁 (Monkey Patching)」的手法，有些人覺得很不嚴謹

Ruby 2.0 之後推出 Refinement 的設計，可以稍微控制一下 Open Class 影響範圍。

###原本
```ruby
class String
  def say_hello
    "hello, #{self}"
  end
end
```

###Refinement
```ruby
module MyHelloString
  refine String do
    def say_hello
      "hello, #{self}"
    end
  end
end
```

變成必須先 `using MyHelloString`

```ruby
using MyHelloString

puts "kitty".say_hello  # => "hello, kitty"
```

參考文件：  
[Ruby 語法放大鏡之「為什麼 2.days.ago 在內建的 irb 會找不到這個方法? 這不是 Ruby 語法嗎?」](http://kaochenlong.com/2016/04/26/open-class/)