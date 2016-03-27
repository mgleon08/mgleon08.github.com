---
layout: post
title: "用 pundit 來做權限管理"
date: 2016-03-26 09:47:16 +0800
comments: true
categories: gem
---

可以搭配 [enum](http://mgleon08.github.io/blog/2016/03/08/enum/) 簡單快速地做出權限管理。

<!--more-->

#指令

產生資料夾 `app/policies/`

```ruby
rails g pundit:install
```

#設定

接著就可以根據你的每個 controller action 去個別設定是否有這權限  
通常會搭配 [devise](http://mgleon08.github.io/blog/2016/01/21/devise-rolify-cancan/) 和 [enum](http://mgleon08.github.io/blog/2016/03/08/enum/)  加上欄位 role 去做調整。


```ruby
class PostPolicy
  attr_reader :user, :post

  def initialize(user, post)
    @user = user
    @post = post
  end

  def update?
    user.admin? or not post.published?
  end
end
```

#Controller

在 controller 就可以藉由 `authorize` 來設定要授權的是什麼

```ruby
def update
  @post = Post.find(params[:id])
  authorize @post
  if @post.update(post_params)
    redirect_to @post
  else
    render :edit
  end
end
```

記得要 `include Pundit`  
也可以加上 `verify_authorized` method 來確保每個 action 都有做好設定

```ruby
class ApplicationController < ActionController::Base
  include Pundit
  after_action :verify_authorized
end
```

#錯誤處理 rescue
可以在 `ApplicationController` 上面直接做處理，相當方便

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery
  include Pundit

  rescue_from Pundit::NotAuthorizedError, with: :user_not_authorized

  private

  def user_not_authorized
    flash[:alert] = "You are not authorized to perform this action."
    redirect_to(request.referrer || root_path)
  end
end
```

#RSPEC

基本上 github 上面就非常清楚了

```ruby
#rails_helper.rb
require "pundit/rspec"
```

```ruby
#spec/policies

describe PostPolicy do
  subject { described_class }

  permissions :update?, :edit? do
    it "denies access if post is published" do
      expect(subject).not_to permit(User.new(admin: false), Post.new(published: true))
    end

    it "grants access if post is published and user is an admin" do
      expect(subject).to permit(User.new(admin: true), Post.new(published: true))
    end

    it "grants access if post is unpublished" do
      expect(subject).to permit(User.new(admin: false), Post.new(published: false))
    end
  end
end
```

gem：
[pundit](https://github.com/elabs/pundit)

參考資料：  
[Rails Notes: Authorization Using Pundit](http://voice.lawrencesun.info/posts/2014/12/13/rails-notes-authorization-using-pundit/)  
[Rail Authorization with pundit and devise](http://www.learning-rails.net/2015/09/rail-authorization-with-pundit-and.html)