---
layout: post
title: "Deploying Rails API + Nuxt.js + Devise-JWT API app to production with Heroku"
date: 2018-07-22 23:12:48 +0800
comments: true
categories: rails
---

接著來把做好的 `Rails API + Nuxt.js + Devise-JWT` deploy 到 heroku

<!-- more -->

* [Deploying your Nuxt+Rails API app to production with Heroku](https://medium.com/@fishpercolator/deploying-your-nuxt-rails-api-app-to-production-with-heroku-54efd09eec22)

由於一開始我們希望用 docker-compose 來 build 環境，因此將前後端都 commit 在一起，但在部屬的時候希望將兩個分開，這時就可以用到 `git subtree`

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com)


# Backend

Rails5.2 必須先將 `production` 的 `config.require_master_key` 打開

```ruby
# autheg-backend/config/environments/production.rb
config.require_master_key = true
```

記得要把 `cors.rb` 設定成 herokuapp 的 domain (根據前端專案的名稱)

```ruby
# autheg-backend/config/initializers/cors.rb
origins 'localhost:3000', 'autheg-frontend-demo.herokuapp.com'
# or
origins '*'
```

安裝 [heroku-cli](https://devcenter.heroku.com/articles/heroku-cli)

```ruby
brew install heroku/brew/heroku
```

create backend 的 heroku 專案

```ruby
# 先到最上層的資料夾
cd autheg
# 如果看到這個 "Name is already taken"，就換一個名字即可
heroku apps:create autheg-backend-demo
```

預設 remote name 會是 heroku，因為等下要在 create frontend remote，因此先改名

```ruby
# 更改 local 的 remote name
git remote rename heroku backend
```

將後端 autheg-backend push 上去

```ruby
# 透過 subtree 先將後端 autheg-backend push 上去
git subtree push --prefix autheg-backend backend master
```

建立環境變數

```ruby
heroku config:set -a autheg-backend-demo RAILS_MASTER_KEY=(local 建立的 config/master.key)

heroku config:set -a autheg-backend-demo JWT_SECRET=$(heroku run -a autheg-backend-demo rails secret)
```

建立 table data

```ruby
heroku run -a autheg-backend-demo rails db:schema:load
```

新增 db 資料


```ruby
heroku run -a autheg-backend-demo rails console

# user
User.create! email: "test@example.com", password: "password"

# example
{"foo" => "green", "bar" => "red", "baz" => "purple"}.each {|n,c| Example.create!(name: n, colour: c)}

```

接下來就可以透過 [YARC](http://yet-another-rest-client.com/) or [Postman](https://www.getpostman.com/) 來測試有沒有成功!

```ruby
POST https://autheg-backend-demo.herokuapp.com/api/users/sign_in

{
   "user":{
      "email":"test@example.com",
      "password":"password"
   }
}
```

# Frontend

[nuxt heroku-deployment](https://github.com/nuxt/docs/blob/master/en/faq/heroku-deployment.md)

create frontend 的 heroku 專案

```ruby
# 先到最上層的資料夾
cd autheg
# 如果看到這個 "Name is already taken"，就換一個名字即可
heroku apps:create autheg-frontend-demo
```

rename remote name

```ruby
git remote rename heroku frontend
```

設定環境變數，要跑在 production 模式，因此要將之前安裝在 dev 環境中的套件，安裝上去

> The following command tells your app to run in production mode and on all interfaces (0.0.0.0) but tells yarn/npm to run in development mode, so that all the dev packages are installed as part of the build process.

```ruby
heroku config:set -a autheg-frontend-demo NODE_ENV=production HOST=0.0.0.0 NPM_CONFIG_PRODUCTION=false
```

改一下 `package.json`，讓 heroku deploy 之後可以 build 檔案出來

```ruby
"heroku-postbuild": "npm run build"
```

設定 backend 的 API 路徑

```ruby
heroku config:set -a autheg-frontend-demo API_URL=https://autheg-backend-demo.herokuapp.com/api
```

接著就可以到首頁上去測試了

```ruby
https://autheg-frontend-demo.herokuapp.com/
# email: test@example.com
# password: password
```

# Heroku
其他 heroku 功能

```ruby
# create new heroku project
heroku create

# push heroku
git push heroku master

# env config remove
heroku config:remove TOKEN

# log
heroku logs --tail

# scale
heroku ps:scale web=2

# ssh
heroku run -a autheg-backend bash

# run 指令
heroku run -a autheg-backend-demo rails console
```

# git subtree

git subtree 本身不支援 force push，因此要透過其他方式來達成

* [publish-ghpages.md](https://gist.github.com/tduarte/eac064b4778711b116bb827f8c9bef7b)

```ruby
git checkout master # you can avoid this line if you are in master...
git subtree split --prefix dist -b gh-pages # create a local gh-pages branch containing the splitted output folder
git push -f origin gh-pages:gh-pages # force the push of the gh-pages branch to the remote gh-pages branch at origin
git branch -D gh-pages # delete the local gh-pages because you will need it: ref
```

參考文件

* [Deploying your Nuxt+Rails API app to production with Heroku](https://medium.com/@fishpercolator/deploying-your-nuxt-rails-api-app-to-production-with-heroku-54efd09eec22)
* [神奇的 Git Subtree](https://hexo.crboy.net/2016/09/amazing-git-subtree/)
* [Git SubTree 共編 Library](http://yutin.logdown.com/posts/188306-git-subtree-total-addendum-library)
* [The Twelve-Factor App](http://erning.net/blog/2012/05/09/the-twelve-factor-app/)
