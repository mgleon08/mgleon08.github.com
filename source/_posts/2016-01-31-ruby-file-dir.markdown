---
layout: post
title: "Ruby - File ＆ Dir 檔案操作"
date: 2016-01-31 19:07:00 +0800
comments: true
categories: ruby
---
用 ruby 來操作 file ＆ Dir 檔案。

<!-- more -->

#File 檔案操作

```ruby
# r  讀取，檔案必須存在
# w  會主動建立空檔案，檔案已存在則會被覆蓋
# a  寫入。主動建檔，檔案已存在則追加在後。
# r+ 讀取 / 寫入，不會主動建檔，將內容加在檔案最前面，會覆蓋原有內容，檔案必須存在)
# w+ 讀取 / 寫入。同 w 功能
# a+ 讀取 / 寫入。同 a 功能

#在每個模式後面加上"b"
#例如 "rb" 或 "rb+"，就可以開啟「二進位」模式

io = File.new("ruby.rb", "w")
io = File.open("/home/work/ruby.rb")
io.close
io.closed? #檢查是否關閉
#若後面沒有 block (結束會自動關閉) 必須要再加上，io.close，關閉檔案，否則會出錯
```

```ruby
# 打開檔案，並寫入文字（若沒檔案會直接新增）
File.open('langs', 'w') do |f|
  f.puts "Ruby"
  f.write "Java\n"
  f << "Python\n"
end
```

```ruby
#查看檔案是否存在，建立時間，檔案大小
puts File.exists? 'tempfile'

f = File.new 'tempfile', 'w'
puts File.mtime 'tempfile'
puts f.size

File.rename 'tempfile', 'tempfile2'

f.close
```

```ruby
#逐一行印出
f = File.open("stones")
while line = f.gets do
    puts line
end
f.close

#逐一行顯示
File.open( "ruby.rb" , "r" ) do |f|
	while line = f.gets
	  puts line
	end
end
```

```ruby
#刪除檔案
File.delete("/home/work/ruby.rb")

#讀取檔案
File.read("ruby.rb")
```

###指標

```ruby
io = File.open("ruby.rb", 'a+')
io.write("123")
#=> 3
io.read
#=> ""

#因為 a 若有檔案在會寫在檔案後面，此時指標會指向最後面，因此可以用 rewind 將指標移致最前面
io.rewind
io.read
#=> 123
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

#\#!/usr/bin/env ruby

在執行檔上加上以下這行，執行時就會知道是要執行 ruby code

```ruby
#!/usr/bin/env ruby

#主要是讓你的程式在不同的系統上都能適用
#不管你的 ruby 是在 /usr/bin/ruby 還是 /usr/local/bin/ruby，會自動的在 PATH 變數中所定義的目錄中尋找ruby來執行

-P參數
#!/usr/bin/env -S -P/usr/local/bin:/usr/bin ruby
#在 /usr/local/bin 和 /usr/bin 目錄下尋找ruby

-S-P參數
#!/usr/bin/env -S-P/usr/local/bin:/usr/bin:${PATH} ruby
#那麼它除了在這兩個目錄尋找之外還會在PATH變數中定義的目錄中尋找
```

#檔案權限

如果要將該檔案變成可執行檔，並且不要讓其他人修改此一檔案的話， 那麼就需要-rwxr-xr-x這樣的權限，此時就得要下達：`chmod 755 test.sh` 的指令囉！

* [檔案權限](http://s2.naes.tn.edu.tw/~kv/file.htm)
* [第五章、Linux 的檔案權限與目錄配置](http://linux.vbird.org/linux_basic/0210filepermission.php)

官方文件：
[Dir](https://ruby-doc.org/core-2.2.0/Dir.html)
[File](http://ruby-doc.org/core-2.3.0/File.html)
[FileUtils](http://ruby-doc.org/stdlib-2.2.2/libdoc/fileutils/rdoc/FileUtils.html#method-c-cd)

參考文件：
[Input & output in Ruby](http://zetcode.com/lang/rubytutorial/io/)
[Ruby#open 知多少？](https://blog.alphacamp.co/2016/06/30/ruby-open/)
[用Ruby Scripting維護系統](http://motion-express.com/trainings/scripting-in-ruby/screencasts/manipulating-files)

