---
layout: post
title: "Import CSV into Rails"
date: 2017-02-14 17:39:16 +0800
comments: true
categories: rails
---

有時候會需要匯入 csv 的檔案, 就可以用 seed 的方式來處理

<!-- more -->

```ruby
#db/seeds/import_csv.rb
require 'csv'

imprt_csv = "#{Rails.root}/lib/seeds/import.csv"

CSV.foreach(csv_text, headers: true) do |row|
  Item.new(name: row[:name])
end
```

檔案可以放在 `lib/csv/xx.csv`

最後用 task 方式去跑 seed 來匯入 [Custom Seed File](http://mgleon08.github.io/blog/2016/07/04/custom-seed-file/)

參考文件：

* [Importing Massive CSV Data Into Rails](https://www.mattboldt.com/importing-massive-data-into-rails/)
* [Ruby on Rails - Import Data from a CSV file](http://stackoverflow.com/questions/4410794/ruby-on-rails-import-data-from-a-csv-file)
* [How to seed a Rails database with a CSV file](https://gist.github.com/arjunvenkat/1115bc41bf395a162084)