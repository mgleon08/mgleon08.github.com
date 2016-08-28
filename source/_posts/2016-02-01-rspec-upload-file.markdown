---
layout: post
title: "如何測試上傳檔案 Rspec upload file"
date: 2016-02-01 21:32:55 +0800
comments: true
categories: rails rspec
---

rails 本身就有內建的 helper 可以很快的建造假的檔案，然後就可以測試上傳的功能了。

<!-- more -->

#建立資料夾

`spec/fixtures/files`

在資料夾裡放入用來測試的檔案

#Rack::Test::UploadedFile.new

`Rack::Test::UploadedFile.new('path','mime-type')`

```ruby
Rack::Test::UploadedFile.new('test.jpg', "image/jpeg")
Rack::Test::UploadedFile.new(File.open(File.join(Rails.root, '/spec/fixtures/files/1.jpg')), "image/jpeg")
```

#fixture_file_upload

上面的簡短版本 `fixture_file_upload('path','mime-type')`

```ruby
@file1 = fixture_file_upload('files/1.jpg', 'image/jpg')
@file2 = fixture_file_upload('files/2.pdf', 'application/pdf')

#若出現 NoMethodError，可加上
include ActionDispatch::TestProcess
```

以上兩個擇一，這樣就可以在 `create` 將 `@file` 帶入到 params


官方文件：  
[Class: Rack::Test::UploadedFile](http://www.rubydoc.info/github/brynary/rack-test/Rack/Test/UploadedFile)  
[fixture_file_upload](http://apidock.com/rails/ActionDispatch/TestProcess/fixture_file_upload)

參考文件：  
[How do I test a file upload in rails?](http://stackoverflow.com/questions/1178587/how-do-i-test-a-file-upload-in-rails)  
[[求助] 文件上传的测试代码怎么写](https://ruby-china.org/topics/21057)