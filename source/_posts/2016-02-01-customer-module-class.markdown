---
layout: post
title: "自定 Module 和 class 檔案"
date: 2016-02-01 22:19:13 +0800
comments: true
categories: rails
---

在 rails 當中，可以自訂一些好用方便的檔案，在適當的時機來使用。

<!-- more -->


#method
在 `lib/require/object.rb`，可以自行新增 class

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

也可以直接在 `config/initializer/require.rb` require 進來，就不用每個檔案上面都 require 了。

```ruby
#單個檔案
require "#{Rails.root}/lib/require/object.rb"

#多個檔案
Dir["#{Rails.root}/lib/require/object.rb"].each do |file|
  require file
end
```

#使用

```ruby
"hello".is?("yes", "no", "hello)
#=> true

"hello".is?("yes", "no")
#=> false
```