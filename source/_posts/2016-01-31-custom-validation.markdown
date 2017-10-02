---
layout: post
title: "自訂 validation"
date: 2016-01-31 12:29:33 +0800
comments: true
categories: rails
---

當內建的驗證沒有自己的需求時，可以自訂驗證的 method 來使用。

<!-- more -->


#Custom method

也可以寫方法來驗證 Model 的狀態，並在 Model 狀態無效的情況下將錯誤加入 errors 集合。必須使用 validate 這個類別方法來註冊。

```ruby
class Picture < ActiveRecord::Base
  belongs_to :user
  validate  :picture_size, on: :create  #注意沒有加 's'

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

#Custom Validations

自定 Validator 是擴展 ActiveModel::Validator 的類別，且必須實作 validate 方法，此方法接受 record 作為參數，驗證行為寫在這個方法裡。寫好 Validator，使用時則是用 validates_with。

###validates_with

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
class PhotoValidator  
  include ActiveModel::Validations
  validates_with PhotoSizeValidator
  
  #也可帶入其他參數
  #validates_with PhotoSizeValidator, fields: [:size]
  #預設會被放入 options Hash，options[:size]
end
```

以上在應用程式生命週期內只會實體化一次，而不是每次驗證時就實體化一次。所以使用實體變數時要很小心。

如果驗證類別足夠複雜的話，需要用到實體變數，可以用純 Ruby 物件（Plain Old Ruby Object, PORO）

###PORO範例
```ruby
class Person < ActiveRecord::Base
  validate do |person|
    GoodnessValidator.new(person).validate
  end
end
 
class GoodnessValidator
  def initialize(person)
    @person = person
  end
 
  def validate
    if some_complex_condition_involving_ivars_and_private_methods?
      @person.errors[:base] << "This person is evil"
    end
  end
 
  # ...
end
```


###validate_each

加入自定 Validator 來驗證每一個屬性的最簡單方法是使用 ActiveModel::EachValidator。在這個例子裡，自定的 Validator 類別必須實作一個 validate_each 方法，接受三個參數，record、attribute 以及 value，分別對應到要驗證的紀錄、屬性、屬性值。

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

這個方法採用區塊（block）來驗證屬性。沒有預先定義的驗證功能。可以在程式碼區塊裡寫要驗證的行為，validates_each 指定的每個屬性，會傳入區塊做驗證。比如下例檢查名與姓是否以小寫字母開頭：

```ruby
class Person < ActiveRecord::Base
  validates_each :name, :surname do |record, attr, value|
    record.errors.add(attr, 'must start with upper case') if value =~ /\A[[:lower:]]/
  end
end
```
這個區塊接受記錄、屬性名稱、屬性值。在區塊裡可以寫任何驗證行為。驗證失敗時應給 Model 新增錯誤訊息，才能把記錄標記成非法的。

#Other
其他要注意的是，並不是每個方法都會去做驗證的動作  
以下方法會忽略驗證

```ruby
decrement!
decrement_counter
increment!
increment_counter
toggle!
touch
update_all
update_attribute
update_column
update_columns
update_counters
```

其他內建驗證直接參考官網

官方文件：  

* [Active Record 驗證](http://guides.rubyonrails.org/active_record_validations.html)  
* [Active Record 驗證 中文](http://rails.ruby.tw/active_record_validations.html)

參考文件：  

* [ActiveRecord - 資料驗證及回呼](https://ihower.tw/rails4/activerecord-lifecycle.html)
* [為你學的 ruby on rails - model 驗證及回呼](http://railsbook.tw/chapters/19-model-validation-and-callback.html)