---
layout: post
title: "Ruby Tips - MultiIO logger"
date: 2016-10-17 17:47:38 +0800
comments: true
categories: ruby ruby_tips
---

logger 可以方便我們去找尋問題在哪邊，因此設定好 logger 訊息是非常重要的

<!-- more -->

在 ruby 中，可以自定義 logger 的 level 還有 format，和輸出方式等等

```ruby
#設定 logger 的輸出
logger = Logger.new(STDOUT)

#設定 logger level
logger.level = Logger::INFO

#設定 logger formatter
logger.formatter = proc do |severity, datetime, progname, msg|
  "#{msg}\n"
end
```

```ruby
# Message in a block.
logger.fatal { "Argument 'foo' not given." }
# Message as a string.
logger.error "Argument #{@foo} mismatch."
# With progname.
logger.info('initialize') { "Initializing..." }
# With severity.
logger.add(Logger::FATAL) { 'Fatal error!' }
```

# MultiIO

同時輸出到 terminal & file

```ruby
class Logger < ::Logger
  class << self
    def default(log)
      # 設定兩種要輸出的 io 方式
      io = [STDOUT, log_file]
      logger = Logger.new(MultiIO.new(*io))
      # 設定 log 層級
      logger.level = Logger::INFO
      # 設定每次輸出 log 的格式
      logger.formatter = proc do |_severity, _datetime, _progname, msg|
        "#{msg}\n"
      end
      logger
    end

    def log_file
      time = Time.now.strftime('%Y-%m-%dT%H:%M:%S')
      FileUtils.mkdir_p('./sample')
      log_file = File.open("./sample/#{time}.log", 'a')
      log_file
    end
  end
end
```

```ruby
class MultiIO
  def initialize(*targets)
    @targets = targets
  end

  def write(args)
  	# 每個 io 都寫入
    @targets.each { |target| target.write(args) }
  end

  def close
    # 寫入完要 close
    @targets.each(&:close)
  end
end
```


官方文件:

* [logger](https://ruby-doc.org/stdlib-2.1.0/libdoc/logger/rdoc/Logger.html)
* [rails guide](http://rails.ruby.tw/debugging_rails_applications.html#logger)

參考文件:

* [How can I have ruby logger log output to stdout as well as file?](http://stackoverflow.com/questions/6407141/how-can-i-have-ruby-logger-log-output-to-stdout-as-well-as-file)
* [How to format ruby logger?](http://stackoverflow.com/questions/14382252/how-to-format-ruby-logger)
* [All Rubyists Love Logging](https://www.sitepoint.com/rubyists-love-logging/)
