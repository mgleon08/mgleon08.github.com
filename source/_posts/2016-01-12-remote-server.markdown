---
layout: post
title: "遠端 server 指令"
date: 2016-01-12 23:02:38 +0800
comments: true
categories: 
---

當部署上 server 後，很多時候都要在 server 上另外打指令  
但可能跟本地的會有稍稍不一樣，所以主要來記錄一下哪些指令

<!--more-->

#MySQL

`mysql -u root -p （輸入後會進入 mysql 的 console ）`

mysql 指令參數

`-h` <- mysql server 位置 , 若沒有指定, 預設就是 localhost  
`-u` <- mysql user 帳號  
`-p` <- 輸入 mysql user 密碼  

若先前已經設定密碼

`mysqladmin -h mysql_server -u root -p password 新密碼`

若是還沒有設定

`mysqladmin -h mysql_server -u root password 新密碼`

`show databases`
可以看目前所有的資料庫

#看遠端log?

nginx log 在 /opt/nginx/logs 下，要用 root 身分才能看

顯示最後500行log  
`tail -n 500 log/production.log`

直掛著顯示log  
`tail -f log/production.log`


#在遠端如何進 rails console?

`cd ~/your_project/current`  
`bin/rails c production` 或 `bundle exec rails c production`


#重開 nginx sudo
`service nginx restart`

#重開遠端 rails 而不重開 nginx
在遠端 `~/your_project/current` 下執行 `touch tmp/restart.txt`

#在遠端跑 rake
`RAILS_ENV=production bin/rake db:seed`

###siedekiq
Sidekiq
`RAILS_ENV=production bundle exec sidekiq -q default -q mailers`
 
重新啟動  
`cap production sidekiq:start`

監控 sidekiq 看有沒有死掉  
[monit](https://mmonit.com/monit/)  
###sitemap
`RAILS_ENV=production rake -s sitemap:refresh`
Sitemap
`RAILS_ENV=production bin/rake sitemap:refresh:no_ping`

#看有沒有定期執行的程式
`crontab -e`