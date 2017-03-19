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

##other

也可以寫在 database.yml 裡面

```ruby
development_db2:
  database: your_database_name_development
  <<: *default

test_db2:
  database: your_database_name_test
  <<: *default
    
production_db2:
  database: your_database_name
  <<: *default
```

```ruby
class OtherDatabase < ActiveRecord::Base
  #記得要用 symbol
  establish_connection "#{RAILS_ENV}_db2".to_sym
end

class User < OtherDatabase
	self.table_name = :user
end
```

其他文件:

* [ActiveRecord in pure Ruby（+ API server）](http://railsfun.tw/t/activerecord-in-pure-ruby-api-server/161)
* [Ruby on Rails Connect to Multiple Databases and Migrations](http://geekhmer.github.io/blog/2015/02/07/ruby-on-rails-connect-to-multiple-databases-and-migrations/)
* [Connecting Models To Different Databases In Rails](https://solidfoundationwebdev.com/blog/posts/connecting-models-to-different-databases-in-rails)
* [How do i work with two different databases in rails with active records?](http://stackoverflow.com/questions/1226182/how-do-i-work-with-two-different-databases-in-rails-with-active-records)
* [Connecting Rails 3.1 with Multiple Databases](http://stackoverflow.com/questions/6122508/connecting-rails-3-1-with-multiple-databases)