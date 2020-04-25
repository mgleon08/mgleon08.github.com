---
layout: post
title: "Github Actions"
date: 2020-04-25 00:30:09 +0800
comments: true
categories: CI
---

<!-- more -->

除了常見的 [travis-ci](https://travis-ci.org/) [circleci](https://circleci.com/) github 也出了自己的 github actions，比較特別的是，針對這些 action 還出了一個 [github action marketplace](https://github.com/marketplace?type=actions)，可以直接拿別人寫好的 action 就可以使用!蠻方便的

以下為 rails + mysql rspec 範例

# Example

```yaml
name: Continuous Integration

# 在什麼時機觸發跑這個 job，也可以針對不同 branch, tag 甚至設定 cronjob 之類的
on: [push]

# 設定環境的參數，記得密碼那些要跟專案一樣
env:
  RUBY_VERSION: 2.6.x
  NODE_VERSION: 12.x
  RAILS_ENV: test
  MYSQL_ROOT_PASSWORD: password
  MYSQL_PORT: 3306

jobs:
  rspec-test:
    name: RSpec
    # 跑在什麼系統 window, ubuntu, mac..
    runs-on: ubuntu-latest

	# 建立專案需要用到的其他 services e.g. mysql, redis ...
    services:
      mysql:
        image: mysql:5.7.29
        env:
          MYSQL_ROOT_PASSWORD: ${{ env.MYSQL_ROOT_PASSWORD }}
          MYSQL_PORT: ${{ env.MYSQL_PORT }}
        ports:
          - 3306:3306
        options: >-
          --health-cmd "mysqladmin ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      # checkout 到指定要跑的 branch 或 tag (一定要加)
      - name: Check out code
        uses: actions/checkout@v2

      - name: Set up Ruby
        uses: actions/setup-ruby@v1
        with:
          ruby-version: ${{ env.RUBY_VERSION }}

      - name: Set up Node
        uses: actions/setup-node@master
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: Install dependencies
        run: |
          gem install bundler -v 1.17.3
          bundle install --jobs 4 --retry 3
          yarn install

      - name: Create database
        env:
          RAILS_ENV: ${{ env.RAILS_ENV }}
          MYSQL_PORT: ${{ env.MYSQL_PORT }}
        run: |
          bundle exec rails db:create
          bundle exec rails db:migrate

      - name: Run Rspec
        run: bundle exec rspec
	   # 如果有加上 simplecov 就會產生一份報告，就可以透過這個別人寫的 action，將檔案產生，之後就可以在跑 action 的地方看到這份檔案
      - name: Upload coverage results
        uses: actions/upload-artifact@master
        if: always()
        with:
          name: coverage-report
          path: coverage
```

Reference:

* [Git Actions](https://github.com/features/actions)
* [Workflow syntax for GitHub Actions](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions)
