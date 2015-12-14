---
layout: post
title: "Ruby on rails - delegate"
date: 2015-12-13 12:25:16 +0800
comments: true
categories: rails語法 rails
---
神奇語法delegate，之前就看過這個語法，不過一直搞不懂在幹什麼？  
研究了之後，發現是個很magic的用法!!

<!-- more -->

>說穿了其實是讓 class 的 object 可以直接呼叫到另一個 class 的 method

還是看範例比較快

```ruby
class Cart < ActiveRecord::Base
  has_many :cart_items
  delegate :empty?, :clear, to: :line_items
end
```
```ruby
class CartItem < ActiveRecord::Base
  belongs_to :cart
end
```

上面有兩個 model 的關係是 one-to-many  
一般正常來說必須要`@cart.cart_items.empty?` 才能夠從 cart 關聯到 cart_items 在呼叫他的方法 `empty?`

不過因為加上了  

```ruby
delegate :empty?, :clear, to: :line_items
```  

所以直接呼叫 `@cart.empty?` 就可以回傳 `@cart.cart_items.empty?`  
仔細觀察後，其實也就等於下面的 method (但是可以一行解決下面的語法  

```ruby
def empty?
    cart_items.empty?
end
```

另外還可以加上 `prefix: true`  

```ruby
delegate :empty?, :clear, to: :line_items, prefix: true
```

變成直接呼叫 `@cart.cart_items_empty?` 字面上面的更加清楚!

在官方API文件上甚至可以看到
可以加上 `allow_nil: true` 允許 nil
甚至是 prefix 可以加上別的字，變成

```ruby
delegate :empty?, :clear, to: :line_items, prefix: mycart
```
就變成 `@cart. mycart_empty?`  
蠻酷的吧!

官方文件：
[delegate](http://apidock.com/rails/Module/delegate)

1. :to - Specifies the target object
2. :prefix - Prefixes the new method with the target name or a custom prefix
3. :allow_nil - if set to true, prevents a NoMethodError to be raised

