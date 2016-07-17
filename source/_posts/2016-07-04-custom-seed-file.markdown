---
layout: post
title: "Custom Seed File"
date: 2016-07-04 22:27:18 +0800
comments: true
categories: rails
---
在 `db` 下有個 seed.rb，可以寫一些假資料 or 要匯入的資料，在執行 `rake db:seed` 就會執行了  
但是當有多個檔案，就可以自己寫個 `task` 來覆蓋原本的 `rake db:seed`

<!-- more --> 

```ruby
#檔案可以放在 db/seeds 資料夾底下
#db/seeds/hello.rb
puts 'hello'

#lib/tasks/custom_seed.rake
namespace :db do
  namespace :seed do
    Dir[File.join(Rails.root, 'db', 'seeds', '*.rb')].each do |filename|
      task_name = File.basename(filename, '.rb').intern    
      task task_name => :environment do
        load(filename) if File.exist?(filename)
      end
    end
  end
end
```

接著執行

```ruby
rake db:seed:hello
#=>hello
```

參考文件：   
[Adding a custom seed file](http://stackoverflow.com/questions/19872271/adding-a-custom-seed-file)