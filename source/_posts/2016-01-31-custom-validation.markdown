---
layout: post
title: "自訂 validation"
date: 2016-01-31 12:29:33 +0800
comments: true
categories: rails
---

當內建的驗證沒有自己的需求時，可以自訂驗證的 method 來使用。

<!-- more -->

```ruby
class Picture < ActiveRecord::Base
  belongs_to :user
  validate  :picture_size  #注意沒有加 's'

  private

  # 驗證圖片的大小
  def picture_size
    if picture.size > 5.megabytes
      errors.add(:picture, "should be less than 5MB")
      #errors[:picture] << "should be less than 5MB"
    end
  end
end
```

#Performing Custom Validations

也可以將 `validator` 拉出來，比較乾淨。

```ruby
#app/validators/photo_size_validator.rb

class PhotoSizeValidator < ActiveModel::Validator
  def validate record
    unless validated? record
      record.errors[:file_size] << 'must be less than 1 MB'
    end
  end

  private

  def validated? record
    record.file.size < 1.megabytes
  end
end

#models/photo.rb
class PhotoValidator < ActiveModel::EachValidator
  include ActiveModel::Validations
  validates_with PhotoSizeValidator
end
```

or

```ruby
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
      record.errors[attribute] << (options[:message] || "is not an email")
    end
  end
end
 
class Person < ActiveRecord::Base
  validates :email, presence: true, email: true
end
```


官方文件：  
[Active Record 驗證](http://guides.rubyonrails.org/active_record_validations.html)  
[Active Record 驗證 中文](http://rails.ruby.tw/active_record_validations.html)

參考文件：  
[ActiveRecord - 資料驗證及回呼](https://ihower.tw/rails4/activerecord-lifecycle.html)