---
layout: post
title: "Ruby Tips - Custom config setting"
date: 2016-10-17 17:36:54 +0800
comments: true
categories: ruby ruby_tips
---
經常看到 rails 中有很多 config 的設定，像是 rspec 等等，會用一個 black 包起來就可以做設定了，那內部實作是如何?

<!-- more -->

```ruby
module Mail
  class Configuration
    DEFAULT_VERSION  = 'v1'
    DEFAULT_API_KEY  = 'token'

    class << self
      attr_writer :version, :api_key

      def configure(&block)
        yield self
        self
      end

      def version
        @version ||= DEFAULT_VERSION
      end

      def api_key
        @api_key ||= DEFAULT_API_KEY
      end
    end
  end
end

Mail::Configuration.configure do |config|
  config.version = 1
  config.api_key = 2
end
```

或是另外一種寫法

```ruby
module Mail
  module Config
    extend self

    attr_accessor :token
    attr_accessor :logger

    def reset
      self.token = nil
      self.logger = nil
    end

    reset
  end

  class << self
    def configure
      block_given? ? yield(Config) : Config
    end

    def config
      Config
    end
  end
end

Mail.configure do |config|
  config.token = ENV['MAIL_API_TOKEN']
  config.logger = Logger.new(STDOUT)
  config.logger.level = Logger::INFO
  fail 'Missing ENV[MAIL_API_TOKEN]!' unless config.token
end
```
