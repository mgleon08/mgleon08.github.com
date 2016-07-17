---
layout: post
title: "用 ruby 進行 File ＆ Dir 檔案操作"
date: 2016-01-31 19:07:00 +0800
comments: true
categories: ruby
---
用 ruby 來操作 file ＆ Dir 檔案。

<!-- more -->
#File 檔案操作

```ruby
File.new("ruby.rb", "w")

# w 覆寫整個檔案
# a 將內容加到檔案後方
# r 將內容加在檔案最前面

File.open("/home/work/ruby.rb")

File.open("ruby.rb", 'w') { |f|
  f.write("puts 'hello world'")
}
# 打開檔案，並寫入文字（若沒檔案會直接新增）

File.delete("/home/work/ruby.rb")
#刪除檔案

File.read("ruby.rb")
#讀取檔案
```

#Path

```ruby
File.expand_path("..", __FILE__)
# => 目前檔案路徑
# __FILE__ 是一個預先定義好的變數，內容是目前這個檔案的名稱

File.join(Rails.root, "ruby.rb")
# => root_path/ruby.rb
```

[What does __FILE__ mean in Ruby?](http://stackoverflow.com/questions/224379/what-does-file-mean-in-ruby)

#File data

```ruby
File.extname("ruby.rb"")
#=> ".rb"

File.basename("/home/work/ruby.rb", ".*")
#=> "ruby"
# `*` Regular Expression，代表不管任何字

File.dirname("/home/work/ruby.rb")
# => "/home/work"
```

#資料夾操作

```ruby
Dir.mkdir("dir")
# 建立資料夾

Dir.entries(".")
# 顯示目前的資料夾的檔案，回傳一個Array

Dir.entries(".").each do |file|
  puts File.file?(file)
end
# 檢查整個資料夾內的項目是否為檔案

Dir["#{Rails.root}/test/*.rb"].each do |file|
  require file
end
```

#判斷
```ruby
File.file?(file)
# 判斷是否為檔案？

File.directory? "dir"
# 判斷是否為資料夾？
```

#FileUtils

移動檔案

```ruby
require 'fileutils'

FileUtils.mv(current_path, other_path)

FileUtils.rm_rf("dir")
# 刪除資料夾

FileUtils.copy_entry("dir", "new_dir")
# 複製整個資料夾內容
```

#pathname

```ruby
require 'pathname'
file = Pathname("./test.rb").expand_path
File.open(file)

#A Pathname is not a String, but File#open doesn't care, because it can be converted to a Wlename String via its #to_path method.
#File.open 預設會去找 to_path 的 method
```

官方文件：  
[File](http://ruby-doc.org/core-2.3.0/File.html)  
[FileUtils](http://ruby-doc.org/stdlib-2.2.2/libdoc/fileutils/rdoc/FileUtils.html#method-c-cd)

參考文件：  
[用Ruby Scripting維護系統](http://motion-express.com/trainings/scripting-in-ruby/screencasts/manipulating-files)