---
layout: post
title: "用 ruby 來 Calling shell commands"
date: 2016-02-07 14:31:31 +0800
comments: true
categories: ruby
---

可以直接透過 ruby 來執行 commands 的指令。

<!-- more -->

* Kernel#\`, commonly called backticks – `cmd` 

Returns the result of the shell command.

```ruby
value = `echo 'hi'`
value = `#{cmd}`
value.class
#=> String 回傳結果
```

* Built-in syntax, `%x( cmd )`  

Returns the result of the shell command, just like the backticks.

```ruby
value = %x( echo 'hi' )
value = %x[ #{cmd} ]
```

* Kernel# `system`

Return: true if the command was found and ran successfully, false otherwise

```ruby
wasGood = system( "echo 'hi'" )
wasGood = system( cmd )
wasGood.class
=> TrueClass 回傳有沒有成功
```

* Kernel#exec

Return: none, the current process is replaced and never continues

```ruby
exec( "echo 'hi'" )
exec( cmd ) # Note: this will never be reached because of the line above
```

官方文件：  
[Kernel](http://ruby-doc.org/core-2.3.0/Kernel.html)  
[Open3 - 可執行精密的操作](http://ruby-doc.org/stdlib-2.3.0/libdoc/open3/rdoc/Open3.html#method-c-pipeline)

參考文件：   
[Calling shell commands from Ruby](http://stackoverflow.com/questions/2232/calling-shell-commands-from-ruby)  
[6 Ways to Run Shell Commands in Ruby Tuesday](http://tech.natemurray.com/2007/03/ruby-shell-commands.html)