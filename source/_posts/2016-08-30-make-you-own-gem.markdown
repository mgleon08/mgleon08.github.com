---
layout: post
title: "Make You Own Gem"
date: 2016-08-30 10:24:36 +0800
comments: true
categories: ruby gem
---

bundle 本身就有提供可以產生 gem 相關檔案的功能

<!-- more -->

```ruby
bundle gem xxx --mit -t=rspec
#後面可以設定參數，ex: 自動產生 LICENSE.txt, 測試用 rspec or mintest 等等

#xxx/Gemfile                  <= dependency 哪些套件，但基本上裡面是指定到 xxx.gemspec，將相依的套件定義在 xxx.gemspec 即可
#xxx/Rakefile                 <= 發佈和打包的 rake tasks
#xxx/LICENSE.txt              <= 註明 License
#xxx/README.md                <= 說明如何使用它(github上會顯示)
#xxx/.gitignore               <= 不要進 Git 的檔案在這定義
#xxx/xxx.gemspec             <= gem 的 spec
#xxx/lib/xxx.rb              <= gem 裡的 library，主要 code 都是在這裡
#xxx/lib/xxx/version.rb      <= 版本紀錄
```

`xxx.gemspec` gem 裡面的基本設定都會在這邊，需要寫的都會註明 `TODO:`

```ruby
# coding: utf-8
lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'xxx/version'

Gem::Specification.new do |spec|
  spec.name          = "xxx"
  spec.version       = Xxx::VERSION
  spec.authors       = ["TODO: Write your name"]
  spec.email         = ["TODO: Write your email address"]

  spec.summary       = %q{TODO: Write a short summary, because Rubygems requires one.}
  spec.description   = %q{TODO: Write a longer description or delete this line.}
  spec.homepage      = "TODO: Put your gem's website or public repo URL here."
  spec.license       = "MIT"

  # Prevent pushing this gem to RubyGems.org by setting 'allowed_push_host', or
  # delete this section to allow pushing this gem to any host.
  if spec.respond_to?(:metadata)
    spec.metadata['allowed_push_host'] = "TODO: Set to 'http://mygemserver.com'"
  else
    raise "RubyGems 2.0 or newer is required to protect against public gem pushes."
  end

  spec.files         = `git ls-files -z`.split("\x0").reject { |f| f.match(%r{^(test|spec|features)/}) }
  spec.bindir        = "exe"
  spec.executables   = spec.files.grep(%r{^exe/}) { |f| File.basename(f) }
  spec.require_paths = ["lib"]

  spec.add_development_dependency "bundler", "~> 1.11"
  spec.add_development_dependency "rake", "~> 10.0"
  spec.add_development_dependency "rspec", "~> 3.0"
end
```

#Code

`xxx/lib/xxx.rb`

```ruby
require "xxx/version"

module Xxx
  # Your code goes here...
  def self.hi
    puts "Hello, world!"
  end
end
```

#Build

```ruby
gem build xxx.gemspec
#目錄底下會多出此檔 xxx-0.0.1.gem

Successfully built RubyGem
Name: xxx
Version: 0.1.0
File: xxx-0.1.0.gem
```

#Install

```ruby
gem install xxx

$ irb
require 'xxx'
#=> true

Xxx.hi
#=>Hello, world!
```

#發佈到 RubyGems.org

* gem update --system
* 申請帳號 [rubygems](https://rubygems.org/sign_up)
* 發佈

```ruby
gem push xxx-0.0.1.gem
#push 後會要輸入你剛剛的申請的帳號密碼，就ok囉
```

###command-reference
[command-reference](http://guides.rubygems.org/command-reference/)

參考文件：

* [https://rubygems.org/](RubyGems.org)
* [bundle gem](http://bundler.io/v1.12/man/bundle-gem.1.html)
* [MAKE YOUR OWN GEM](http://guides.rubygems.org/make-your-own-gem/)
* [Ruby 建立自己的 Gem 套件](http://blog.jex.tw/blog/2013/11/29/ruby-build-own-gem/)
* [Step-by-Step Guide to Building Your First Ruby Gem](https://quickleft.com/blog/engineering-lunch-series-step-by-step-guide-to-building-your-first-ruby-gem/)