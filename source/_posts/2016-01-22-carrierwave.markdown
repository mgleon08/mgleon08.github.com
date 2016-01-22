---
layout: post
title: "用 carrierwave 輕鬆做上傳檔案功能"
date: 2016-01-22 22:12:46 +0800
comments: true
categories: gem rails
---

另一個上傳檔案的 gem ，相當的實用，和 Paperclip 擇一即可。
<!-- more -->

#安裝
Gemfile

```ruby
gem 'carrierwave'
gem 'rmagick'

# or gem "mini_magick"
```

* carrierwave 上傳檔案
* rmagick 處理圖片

>必須有安裝 ImageMagick 才能使用 rmagick  

有 homebrew 可以直接

```ruby
brew install imagemagick
```

#設定

先建立資料夾，主要用來存放所有的檔案 
 
```ruby
rails generate uploader file
```

接著在在要存放的 model 新增一筆欄位

```ruby
rails generate migration add_file_to_products
```

#Model

```ruby
class Product < ActiveRecord::Base
  mount_uploader :file, FileUploader
end
```

最後記得再 strong params 加入

```ruby
private
  def product_params
    params.require(:product).permit(:file)
  end
end
```

#Form

```ruby
<%= form_for @product do |f| %>
  <%= f.file_field :file %>
  <%= f.submit "Submit" %>
<% end %>
```

#view

```ruby
<%= image_tag @product.image_url.to_s %>
```
imageurl是預設的helper，to_s是要確定把上傳的路徑轉變為字串，以免發生錯誤。


#RMagick
`app/uploaders/file_uploader.rb`

可以將設定打開，有 RMagick 和 MiniMagick ，都是用來縮圖的。(擇一即可)

```ruby
# Include RMagick or MiniMagick support:
# include CarrierWave::RMagick
# include CarrierWave::MiniMagick
```

下面把註解拿掉就可以使用

```ruby
Create different versions of your uploaded files:
version :thumb do
  process :resize_to_fit => [50, 50]
end
```

之後再 view 中只要指定，就會去抓取縮小後的圖檔

```ruby
<%= image_tag @product.image_url(:thumb).to_s %>
```

#其他
限制存取大小  
[How to: Validate attachment file size](https://github.com/carrierwaveuploader/carrierwave/wiki/How-to:-Validate-attachment-file-size)

存取圖片長寬  
[How to: Get image dimensions](https://github.com/carrierwaveuploader/carrierwave/wiki/How-to:-Get-image-dimensions)

存取圖片大小和類型  
[How to: Store the uploaded file size and content type](https://github.com/carrierwaveuploader/carrierwave/wiki/How-to:-Store-the-uploaded-file-size-and-content-type)

官方文件：  
[carrierwave](https://github.com/carrierwaveuploader/carrierwave)

參考資料：  
[Ruby gem 'Carrierwave' 上傳檔案神器的簡易安裝與使用](http://motion-express.com/blog/20140708-ruby-gem-carrierwave)  
[使用 Carrierwave 處理檔案上傳 (整合 imagemagick 與 Amazon S3)](http://rubyist.marsz.tw/blog/2012-01-10/carrierwave-guides-with-amazon-s3-and-imagemagick-integration/)   
[gem 'carrierwave'简易实用介绍](https://ruby-china.org/topics/4992)  
[Rails如何使用Carrierwave上傳圖片](http://springok-blog.logdown.com/posts/2015/10/21/railsgem-how-to-use-carrierwave-upload-pictures)