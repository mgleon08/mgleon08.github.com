---
layout: post
title: "RVM and Gemsets ruby版本控制"
date: 2016-02-15 21:12:09 +0800
comments: true
categories: ruby
---
好用的工具，可以輕鬆的切換 ruby 版本!

<!-- more -->

#安裝

[rvm](https://rvm.io/)

```
\curl -sSL https://get.rvm.io | bash -s stable
```

###安裝 ruby 版本
```
rvm install 2.2.3
```


###使用版本
```
rvm use 2.2.3
```

###目前有的版本
```
rvm list
```

###移除
```
rvm remove 2.2.3
```

###看本機是否使用 rvm 還是本機
```
which ruby
```

#設定

安裝好之後，基本上就是一個全新的，gem 都要全部重新安裝  
記得先去安裝 `bundler`

```
gem install bundler
```

之後再去專案底下輸入 `bundle` 就會重新安裝了


#Gemset

在該版本的 ruby 下，在區分要安裝哪些 gem

###目前有的 gemset
```
rvm gemset list
```

###新增
```
rvm gemset create rails4.2.4
```

###使用
```
rvm gemset use rails4.2.4
```

###清空
清空裡面的 gem ，但保留 gemset
```
rvm gemset empty rails4.2.4
```

###刪除
```
rvm gemset delete rails4.2.4
```

#Bonus

###直接換兩個
```
rvm 2.2.3@rails4.2.4
# ruby 和 gemset 一起切換
```

官方文件：  
[rvm](https://rvm.io/)

參考文件：  
[Ruby gem 想要一機裝多個版本？RVM來幫你！](http://motion-express.com/blog/20141005-ruby-rvm-gemset)  
[RVM and Gemsets](http://blog.eddie.com.tw/2011/04/08/rvm-and-gemsets/)

