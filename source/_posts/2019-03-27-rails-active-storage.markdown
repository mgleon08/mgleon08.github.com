---
layout: post
title: "Rails with Active Storage"
date: 2019-03-27 09:33:19 +0800
comments: true
categories: rails
---

<!-- more -->

`Active Storage` 是 rails 後來出的新功能，可以簡單的做到上傳檔案，雲端整合([Amazon S3](https://aws.amazon.com/tw/s3/), [Google Cloud Storage](https://cloud.google.com/storage/docs/))，`MiniMagick` 影像操作等等，就像是 [carrierwave](https://github.com/carrierwaveuploader/carrierwave), [paperclip](https://github.com/thoughtbot/paperclip) 等比較常用到的套件

跟以往用 carrierwave 或是 paperclip 不太一樣，不需要在現有的 model 新增欄位，而是透過 `polymorphic` 的方式，將檔案存在兩張獨立的 table， `active_storage_blobs`, `active_storage_attachments`

# Init Project

```ruby
rails new active_storage_demo --webpack=stimulus --skip-coffee --skip-test -d=mysql
```

# Install active_storage

透過 `rails active_storage:install` 建立 active_storage 所需要的兩張 table，包括 `active_storage_blobs`, `active_storage_attachments` 

* `active_storage_attachments`: 用來存跟 model 相關的資訊
* `active_storage_blobs`: 用來存詳細檔案的資訊(裡面包含了 `checksum` 將檔案做 hash 可以知道是否為同一個檔案)

接著跑 `rake db:create db:migrate`

```ruby
# This migration comes from active_storage (originally 20170806125915)
class CreateActiveStorageTables < ActiveRecord::Migration[5.2]
  def change
    create_table :active_storage_blobs do |t|
      t.string   :key,        null: false
      t.string   :filename,   null: false
      t.string   :content_type
      t.text     :metadata
      t.bigint   :byte_size,  null: false
      t.string   :checksum,   null: false
      t.datetime :created_at, null: false

      t.index [ :key ], unique: true
    end

    create_table :active_storage_attachments do |t|
      t.string     :name,     null: false
      t.references :record,   null: false, polymorphic: true, index: false
      t.references :blob,     null: false

      t.datetime :created_at, null: false

      t.index [ :record_type, :record_id, :name, :blob_id ], name: "index_active_storage_attachments_uniqueness", unique: true
      t.foreign_key :active_storage_blobs, column: :blob_id
    end
  end
end
```

預設檔案存在 local(也可以改成 s3 或 GCS)

```ruby
# config/environments/development.rb
config.active_storage.service = :local

# config/storage.yml
test:
  service: Disk
  root: <%= Rails.root.join("tmp/storage") %>

local:
  service: Disk
  root: <%= Rails.root.join("storage") %>
```

建立 user moblie

```ruby
rails g scaffold User name
rake db:migrate
```

在 user model 設定可以上傳的檔案有哪些

```ruby
# models/user.rb
class User < ApplicationRecord
  has_one_attached :avatar
  has_many_attached :documents
end
```

在 view 上面新增畫面，如果是要多個檔案，要加上 `multiple: true`

```ruby
# views/users/_form.html.erb

<div class="field">
  <%= form.label :avatar %>
  <%= form.file_field :avatar %>
</div>

<div class="field">
  <%= form.label :documents %>
  <%= form.file_field :documents, multiple: true %>
</div>
```

* `image_tag(@user.avatar)` 顯示圖片
* `rails_blob_path(doc, disposition: "attachment"))` 附件

```ruby
# views/users/show.html.erb
<h2>avatar</h2>
<% if @user.avatar.attached? %>
  <%= image_tag(@user.avatar) %>
<% end %>

</p>

<% if @user.documents.attached? %>
  <p>
  <h2>Download Documents</h2>
  <% @user.documents.each_with_index do |doc, i| %>
    <p>
    <%= link_to("Document #{i}", rails_blob_path(doc, disposition: "attachment")) %>
    </p>
  <% end %>
  </p>
<% end %>
```

並在 controller 新增 permit，因為 documents 是多個檔案，所以給 Array

```ruby
# controllers/users_controller.rb
def user_params
  params.require(:user).permit(:name, :avatar, documents: [])
end
```

接著就可以 `rails s` 來新增 user 測試看看了

# 刪除檔案

```ruby
# 同步刪除頭像和實際資源檔案。
@user.avatar.purge

# 透過 Active Job 非同步刪除相關模型和實際資源檔案。
@user.avatar.purge_later
```

# 調整圖片尺寸

不同尺寸可以透過 [minimagick](https://github.com/minimagick/minimagick) 做轉換

```ruby
# Gemfile
gem 'mini_magick'

# views/users/show.html.erb
<%= image_tag @user.avatar.variant(resize: "100x100") %>
```

# Direct Upload

```ruby
yarn add activestorage
```


```ruby
# javascript/packs/direct_upload.js

import * as ActiveStorage from 'activestorage';
ActiveStorage.start();
```

# other

```ruby
# url
url_for(@user.avatar)

# download link
rails_blob_path(user.avatar, disposition: "attachment")

Rails.application.routes.url_helpers.rails_blob_url(@user.avatar, only_path: true)
```

# 後續

接著看一下 source code，可以發現到 blob 可以有多個 attachments。

代表 blob 是可以重複利用，但是必須在 `has_one/many_attached` 後面加上 `dependent: false` 才不會刪除了一個 attachment 就將 blob 刪除，導致其他 attachment 關聯不到

另外 [delegate_missing_to](https://www.rubydoc.info/github/rails/rails/Module:delegate_missing_to) 則是類似 delegate 的加強版，讓 user 可以直接 call `user.filename`

> [Rails 5.1 add delegate_missing_to](https://ruby-china.org/topics/33119)

`belongs_to :record` 則是透過 `polymorphic` 的方式來存取是由哪個 model 的資料

```ruby
# rails/activestorage/app/models/active_storage/blob.rb
class ActiveStorage::Blob < ActiveRecord::Base
  # ...
  self.table_name = "active_storage_blobs"
  has_many :attachments
  # ...
 end
```

```ruby
# rails/activestorage/app/models/active_storage/attachment.rb
class ActiveStorage::Attachment < ActiveRecord::Base
  self.table_name = "active_storage_attachments"
  belongs_to :record, polymorphic: true, touch: true
  belongs_to :blob, class_name: "ActiveStorage::Blob"
  delegate_missing_to :blob
end
```

# 缺點

* 有人提到 Active Storage 目前不適合用在 production [Active Storage 的正確使用姿勢](https://ruby-china.org/topics/37717)
* 也有人提到，容易造成 N+1，永久網址難以 cache，redirect 效能問題，routing 衝突等等 [Active Storage 開箱文](https://5xruby.tw/posts/active-storage-review/)

# demo

[active_storage_demo](https://github.com/mgleon08/active_storage_demo)

Reference:

* [Active Storage Overview](https://guides.rubyonrails.org/active_storage_overview.html)
* [activestorage](https://github.com/rails/rails/tree/master/activestorage)
* [Active Storage after Rails 5.2](https://medium.com/@ebifurai_tsn/active-storage-after-rails-5-2-26ad89c93b57)
* [手把手學習使用 Rails 5.2 ActiveStorage (DirectUpload + ProgressBar)](https://andyyou.github.io/2018/06/26/using-active-storage/)
