---
layout: post
title: "在專案中連接另一個 database"
date: 2016-10-24 10:17:44 +0800
comments: true
categories: rails
---

有時候會需要連接另外一個 database 的資料，rails 可以很方便地來做連接!

<!-- more -->

在 model 底下建立子資料夾，連接到另外一個 database，不過關聯就需要另外再去定義，因為是單純去 database 去拉資料過來

* 好處：就不用透過 api 去拉資料
* 壞處：當拉的 database，結構有變就必須跟著動到 code

```ruby
#models/test/base.rb
class Test::Base < ActiveRecord::Base
  self.abstract_class = true
  ActiveRecord::Base.establish_connection(
    :adapter  => "mysql2",
    :host     => "localhost",
    :username => "root",
    :password => "",
    :database => "database_name"
  )
  end
```

```ruby
#models/test/course.rb
class Test::Course < Test::Base
  self.table_name = :Course
  has_many :teachers, class_name: "Test::Teacher"
end
```

```ruby
#models/test/teacher.rb
class Test::Teacher < Test::Base
  self.table_name = :teacher
  belongs_to :course, class_name: "Test::Course"
end
```

其他文件:

* [ActiveRecord in pure Ruby（+ API server）](http://railsfun.tw/t/activerecord-in-pure-ruby-api-server/161)