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

```ruby
\curl -sSL https://get.rvm.io | bash -s stable
```

### 安裝 ruby 版本

```ruby
rvm install 2.2.3
```


### 使用版本(該 terminal)

```ruby
rvm use 2.2.3
# use 可省略 rvm 2.2.3
```

### 設定 default 使用版本

```ruby
rvm use 2.4.0 --default
# rvm 2.4.0 --default
```

### 切回到原來系統內建的版本

```ruby
rvm system
```

### 目前有的版本

```ruby
rvm list
```

### 查目前可以安裝的版本

```ruby
rvm list known
```

### 移除

```ruby
rvm remove 2.2.3
```

### 看本機是否使用 rvm 還是本機

```ruby
which ruby
```

### update rvm

```ruby
rvm get stable
#master     - install the master release
#stable     - install the latest RVM stable release
#latest     - install the latest RVM release
```

# 設定

安裝好之後，基本上就是一個全新的，gem 都要全部重新安裝
記得先去安裝 `bundler`

```ruby
gem install bundler
```

之後再去專案底下輸入 `bundle` 就會重新安裝了


#Gemset

在該版本的 ruby 下，在區分要安裝哪些 gem

###目前有的 gemset
```ruby
rvm gemset list
```

###新增
```ruby
rvm gemset create rails4.2.4
```

###使用
```ruby
rvm gemset use rails4.2.4
```

使用 default

```ruby
rvm gemset use default
```

###清空
清空裡面的 gem ，但保留 gemset
```ruby
rvm gemset empty rails4.2.4
```

###刪除
```ruby
rvm gemset delete rails4.2.4
```

### 自動載入 gemset

```ruby
#建立一個 .rvmrc 文件。
.rvmrc

#在這個文件裡可以很簡單的加一個命令：
rvm use 1.9.3@rails313
#然後無論你當前 Ruby 設定是什麼，cd 到這個專案的時候，RVM 會幫你載入 Ruby 1.9.3 和 rails313 gemset.
```

#Bonus

###直接換兩個

```ruby
rvm 2.2.3@rails4.2.4
# ruby 和 gemset 一起切換

rvm --default use ruby-1.9.3-p0@rails3.2
#設定好兩個的 default 值
```

#other
另一個版本控制是 `rbenv` 聽說比較輕量
project 要加上 `.ruby-version` 裡面寫 ruby 版本

```ruby
2.2.3
```
才能加入控管


#openssl

`rvm install` 新的 ruby 版本的時候，rvm 會先自行去找編譯好的 ruby 版本

* [rvm binaries](https://rvm_io.global.ssl.fastly.net/binaries/)

但若沒有對應的版本，就會直接編譯，但這時候很容易沒包到 openssl

```ruby
# 安裝 openssl
brew install openssl

# 看 openssl 位置
brew info openssl

# 安裝 ruby 並且指定 openssl 位置
rvm install 2.4.0 --with-openssl-dir=`brew --prefix openssl`

# 檢查 ssl
rvm osx-ssl-certs update all

# Tells rvm to automatically use the OpenSSL library installed by Homebrew
rvm autolibs enable

# 查看 rvm 需要 require 什麼
rvm requirements

# 其他的指令
rvm help
```

另一個方法是使用 rvm 內建的 openssl

```ruby
#要先解壓縮
rvm pkg install openssl
rvm remove 2.4.0
rvm install 2.4.0 --with-openssl-dir=$HOME/.rvm/usr
```

openssl 解決方式參考:

* [RVM/Rbenv RVM 安裝 Ruby 2.0.0 的 OpenSSL 問題 ](https://ruby-china.org/topics/8589)
* [troubles with RVM and OpenSSL](http://stackoverflow.com/questions/15511943/troubles-with-rvm-and-openssl)
* [Fixing SSL errors in rvm for OSX](https://schneid.io/blog/fixing-ssl-errors-in-rvm-for-osx.html)
* [SSL_connect返回= 1 errno = 0 state = SSLv3讀伺服器證書B：證書驗證失敗](https://gxnotes.com/article/21594.html)
* [SSL upgrades on rubygems.org and RubyInstaller versions](https://gist.github.com/luislavena/f064211759ee0f806c88)

官方文件：
[rvm](https://rvm.io/)

參考文件：
[Ruby gem 想要一機裝多個版本？RVM來幫你！](http://motion-express.com/blog/20141005-ruby-rvm-gemset)
[RVM and Gemsets](http://blog.eddie.com.tw/2011/04/08/rvm-and-gemsets/)
[rvm 使用指南](https://ruby-china.org/wiki/rvm-guide)

