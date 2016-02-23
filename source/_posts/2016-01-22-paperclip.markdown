---
layout: post
title: "用 paperclip 輕鬆做上傳檔案功能"
date: 2016-01-22 22:12:20 +0800
comments: true
categories: gem rails
---

在網站中經常會需要用到上傳檔案功能
而 Paperclip 就是可以在上傳檔案這件事變得更加便利。

<!-- more -->

#安裝

```ruby
gem "paperclip"
```

#設定

```ruby
class AddAvatarColumnsToUsers < ActiveRecord::Migration
  def up
    add_attachment :topics, :picture
  end

  def down
    remove_attachment :topics, :picture
  end
end
```

attachment 會自動產生以下四個攔位

```ruby
t.string   "picture_file_name",    limit: 255
t.string   "picture_content_type", limit: 255
t.integer  "picture_file_size",    limit: 4
t.datetime "picture_updated_at"
```

#Model

```ruby
class User < ActiveRecord::Base
  has_attached_file :picture, styles: { medium: "300x300>", thumb: "100x100>" }, default_url: "/images/:style/missing.png"
  validates_attachment_content_type :picture, content_type: /\Aimage\/.*\Z/
end
```

#Form

```ruby
<%= form_for @user, url: users_path, html: { multipart: true } do |form| %>
  <%= form.file_field :picture %>
  <%=%>
  
<% if @user.picture.exist?%>
	<%= check_box_tag “remove” ,”1”%> #刪除按鈕
<%end%>
<% end %>
```

在編輯的時候設定如果remove=1就設定圖片欄位為nil就會刪除

###controller

```ruby
if params[:remove]==1
    @user.picture=nil
end
```

#View

```ruby
<%= image_tag @topic.picture.url %>
<%= image_tag @topic.picture.url(:medium) %>
<%= image_tag @topic.picture.url(:thumb) %>
```

#同時上傳多個檔案

1-to-1 可以使用 `fields_for` 來達成，但若是 1-to-many ，則必須要搭配 JavaScript 協助動態增減欄位，動態加減數量。  

可以使用以下 gem  
[nested_form_fields](https://github.com/ncri/nested_form_fields)  

(注意 Strong Parameter，這個 gem 的 README 沒提到)

###model

```ruby
has_many :banners, dependent: :destroy
accepts_nested_attributes_for :banners, allow_destroy: true, :reject_if => :all_blank
```

###strong params

```ruby
params.require(:company).permit(:banners_attributes => [:id, :banner, :banner_alt, :_destroy])
```

###form

```ruby
<div class="image">
  <p>Banner</p>
    <%= c.nested_fields_for :banners do |b| %>
      <div class="form-group">
      <%= b.file_field :banner%>
      <%= b.label :banner_alt, "banner_alt" %><br>
      <%= b.text_field :banner_alt, :class=>"form-control"%>
      <%= b.remove_nested_fields_link 'Remove me'%>
    <% end %>
    <%=c.add_nested_fields_link :banners, '新增Banner', :class => "btn btn-default"%>
</div>

```

或是  [cocoon](https://github.com/nathanvda/cocoon)

###可參考之前的  
[Ruby on Rails - Accepts_nested_attributes_for](http://mgleon08.github.io/blog/2015/12/13/ruby-on-rails-accepts-nested-attributes-for/)


官方文件：  
[paperclip](https://github.com/thoughtbot/paperclip)  
[nested_form_fields](https://github.com/ncri/nested_form_fields)  
[cocoon](https://github.com/nathanvda/cocoon)  

參考文件：  
[使用 paperclip 實作任意格式檔案上傳](http://chouandy.logdown.com/posts/249554-use-paperclip-implement-any-format-file-uploading)  
[使用 paperclip 實作任意格式檔案上傳至 AWS S3](http://chouandy.logdown.com/posts/252165-use-paperclip-implement-any-format-file-uploading-to-aws-s3)  
[Rails 上傳 Upload](http://blog.jex.tw/blog/2015/07/13/rails-upload/)  
[Multiple files upload with nested resource using Paperclip in Rails](http://www.railscook.com/recipes/multiple-files-upload-with-nested-resource-using-paperclip-in-rails/)  
[Paperclip 圖片上傳與額外說明](http://railsfun.tw/t/paperclip/64)
