---
layout: post
title: "Environment banner"
date: 2018-07-23 21:35:24 +0800
comments: true
categories: rails
---

可以很快速的知道目前環境上的版本是什麼

<!-- more -->

公司因為多台機器不同環境，因此有同事做了一個 `Environment banner`，可以很方便地知道目前是哪一個 branch 哪一個 版本，這邊就來記錄一下

```ruby
# app/helpers/environment_banner_helper.rb
module EnvironmentBannerHelper
  RELEASE_INFO_PATH = Rails.public_path.join('release_info')

  def current_branch
    if git_available?
      # 讀取當前 HEAD 所在的 branch 名稱
      `git rev-parse --abbrev-ref HEAD`.chomp
    else
      # 當在 deploy 的時候，無法用 git (除非另外裝)，因此可以先將資訊存放在某的檔案，或是環境變數
      git_info[:branch] # 檔案
      # ENV.fetch("CURRENT_BRANCH", "--branch-not-found--") # ENV
    end
  end

  def current_sha
    if git_available?
      # 讀取當前的 log 最新一行
      `git log --oneline -1`
    else
      # 當在 deploy 的時候，無法用 git (除非另外裝)，因此可以先將資訊存放在某的檔案，或是環境變數
      git_info[:sha] # 檔案
      # ENV.fetch("CURRENT_SHA", "--sha-not-found--") # ENV
    end
  end

  # 確認當前環境有沒有 git
  def git_available?
    to_dev_null = "> /dev/null 2>&1"
    system("which git #{to_dev_null} && git rev-parse --git-dir #{to_dev_null}")
  end

  # 主要是為了 capistrano 上面會有不同的 release 資料夾
  def release_number
    return unless check_release_info && IO.readlines(RELEASE_INFO_PATH)[0]
    IO.readlines(RELEASE_INFO_PATH)[0].gsub(/\D/, '').prepend('#')
  end

  def git_info
    return { sha: "N/A", branch: "N/A"} unless check_release_info && IO.readlines(RELEASE_INFO_PATH)[1]
    line = IO.readlines(RELEASE_INFO_PATH)[1].split(/\s+/)
    {
      sha:    line[0][0..6],
      branch: line[1].gsub(/refs\/heads\//, '')
    }
  end

  def check_release_info
    File.exist? RELEASE_INFO_PATH
  end
end
```

```ruby
# app/views/layouts/application.html.erb
<% unless Rails.env.production? %>
  <div class="environment-banner <%= Rails.env %>">
    <%= Rails.env %> | <%= "#{current_branch} @ #{current_sha} #{release_number}" %>
  </div>
<% end %>
```

```css
/* app/assets/stylesheets/application.scss */
.environment-banner.development {
  background: $green;
  color: $white;
}
```

capistrano deploy 時新增 `release_info` 檔案方式

```ruby
set :repo_url, 'git@github.com:xxx/xxx.git'
ask :branch, `git rev-parse --abbrev-ref HEAD`.chomp

# 將所有 git 撞況寫到一個 file 上，再透過該 file 來顯示
# readlink 可以找到實際檔案位置，awk 分析出路徑上最後的 release 資料夾名稱
execute("echo current_release: `readlink -f #{release_path} | awk -F'/' '{print $NF}'` > #{shared_path.join('public/release_info')}")
# header
execute("git ls-remote -h #{fetch(:repo_url)} #{fetch(:branch)} >> #{shared_path.join('public/release_info')}")
# tag
execute("git ls-remote -t #{fetch(:repo_url)} | tail -n 1 >> #{shared_path.join('public/release_info')}")
```
