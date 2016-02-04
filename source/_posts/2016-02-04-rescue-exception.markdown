---
layout: post
title: "例外處理 rescue exception"
date: 2016-02-04 21:51:38 +0800
comments: true
categories: rails
---

在 rails 當中，當發生例外時就會爆錯，畫面就會不見。  
但有時我們並不希望讓它這樣，因此可以用 rescue 才處理掉這些例外發生時，該執行的動作。

<!-- more -->

#例外處理

```ruby
begin
  # 有可能發生例外的處理動作
rescue => e
  # 例外發生時的處理措施
ensure
  # 無論有沒有發生例外，這一段都一定會執行
end
```

#分開處理例外

可以給予不同例外，執行不同動作

順序應為最特殊為第一位，以此類推  
若要在最後包含所有例外，可以使用rescue Exception

```ruby
begin
  # 有可能發生例外的處理動作
rescue ArgumentError => e
  # 例外發生時的處理措施
rescue TypeError => e
  # 例外發生時的處理措施
rescue Exception => e
  # 例外發生時的處理措施
end
```
如果沒有指定變數，例外物件會自動存放在：`$!`及`$@`變數中  
`$!`：最後發生例外的物件  
`$@`：呈現最後例外所發生的位置和資計


#重來

例外發生後，再重新執行一次

```ruby
begin
  # 有可能發生例外的處理動作
rescue => e
  retry #重新再跑
end
```

#例外語法的簡化

如果例外 begin & end 的範圍剛好就是整個方法的範圍，就可以省略。

```ruby
def rescue
  #有可能發生例外的處理動作
rescue
  #例外發生時的處理措施
ensure
  #無論是否發生例外都會執行
end

```

#自行產生例外

```ruby
def test
  raise StandardError, "test error"
  #丟例外出來 raise(例外名稱, 例外訊息)
rescue => e
  binding.pry
end
```

參考文件：  
[Ruby - Chapter 09 例外處理(exception)](http://blog.xuite.net/yschu/wretch/104912690-Ruby+-+Chapter+09+%E4%BE%8B%E5%A4%96%E8%99%95%E7%90%86(exception))  
[Ruby學習筆記(8) – 錯誤與例外處理](http://blog.tonycube.com/2011/07/ruby8.html)