---
layout: post
title: "OptionParser"
date: 2016-08-30 10:30:52 +0800
comments: true
categories: ruby
---

#OptionParser

ruby 裡內建的強大工具，可以解析命令行參數

<!-- more -->

先來看一下什麼是 Unix命令行

通常的Unix命令行參數包含下面這些形式：

* Option - 主要功能是用於調整命令行工具的行為
	* Option的表現通常有兩種形式
		* short opt​​ion
		* long option 	
	* Option的類型有兩種
		* `switch` 不帶 argument
		* `flag` 帶有 argument
* Argument - 表示命令行工具要操作的對象，通常是路徑，URL或者名稱等等。
* Action - 表示命令行工具的行為，比如 git 命令的 push 或者 pull 等等。

```ruby
git log --max-count=10

#git 是 command，log 是 action，表示查看 git 的提交歷史。
#--max-count 則是 option，表示最多顯示N條commit記錄
```

用 `ARGV` 來接命令列引數的陣列。

```ruby
require 'optparse'

class Hello
  def initialize(arguments)
      @arguments = arguments
      parse_options
    end

  def parse_options
    options = OptionParser.new
    options.banner = 'Usage: xxxx [options]'
    #on('short opt​​ion', 'long option', 'comment')
    #帶參數
    options.on('-a arg', '--aa arg', 'this is test') { |arg| puts arg }
    #不帶參數
    options.on('-b', '--bb', 'this is test') { puts 'b' }
    #多個參數
    options.on('-c arg', '--cc arg', 'this is test') { |arg| puts arg }
    options.on('-h', '--help', 'Show this message') { puts(options); exit }
    options.parse!(@arguments)
  end
end

Hello.new(ARGV)
```

```ruby
ruby optparse.rb -a a
#=>a
ruby optparse.rb --aa aa
#=>aa
ruby optparse.rb -b
#=>b
ruby optparse.rb --bb
#=>b
ruby optparse.rb -c c,cc
#=>c,cc

ruby optparse.rb -h
#Usage: xxxx [options]
#   -t, --test arg                   this is test
#   -h, --help                       Show this message
```


###ARGV

[Learn Ruby The Hard Way ex13](http://lrthw.github.io/ex13/)

```ruby
first, second, third = ARGV
puts "Your first variable is: #{first}"
puts "Your second variable is: #{second}"
puts "Your third variable is: #{third}"


# ARGV 就是「參數變數(argument variable)」，是一個非常標準的程式術語。
# 在其他的程式語言你也可以看到它全大寫的原因是因為它是一個「常數(constant)」，
# 意思是當它被賦值之後你就不應該去改變它了。這個變數會接收當你運行 Ruby 腳本時所傳
# 入的參數。通過後面的習題你將對它有更多的了解。你將對它有更多的了解。

# 第 1 行將 ARGV 「解包(unpack)」，與其將所有參數放到同一個變數下面，我們將每個參數
# 予一個變數名稱 first 、 second 以及 third。腳本本身的名稱被存在一個特殊變數 $0
# 裡，這是我們不需要解包的部份。也許看來有些異，但「解包」可能是最好的描述方式了。它
# 的涵義很簡單：「將 ARGV 中的東西解包，然後將所有
# 的參數依次賦予左邊的變數名稱」。
```

官方文件：  

* [OptionParser](http://ruby-doc.org/stdlib-2.3.1/libdoc/optparse/rdoc/OptionParser.html)

參考文件：  

* [用 OptionParser 构建 Command Line 工具](https://ruby-china.org/wiki/building-a-command-line-tool-with-optionparser)  
* [How do I make a command-line tool in Ruby?](http://rubylearning.com/blog/2011/01/03/how-do-i-make-a-command-line-tool-in-ruby/)

相關 gem： 

* [choice](https://github.com/defunkt/choice)  
* [trollop](https://github.com/ManageIQ/trollop) 