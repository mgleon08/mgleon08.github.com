---
layout: post
title: "會員管理系統 Devise-Rolify-Cancan"
date: 2016-01-21 22:17:48 +0800
comments: true
categories: gem rails
---

Devise + Rolify + Cancancan

Devise    負責登入、註冊、退出等等，會員註冊登入流程  
Rolify    負責給予角色  
Cancancan 負責指定角色的權限，可以執行哪些 action

<!-- more -->

```
gem 'devise'
gem 'rolify'
gem 'cancancan'
```

#[Device](https://github.com/plataformatec/devise)
```ruby
rails generate devise:install

#create  config/initializers/devise.rb
#create  config/locales/devise.en.yml
```

###接下來有幾個設定是 divice 建議設定的

* 確認 root 有設定到

```ruby
root "welcome#index"
```

* 編輯 `config/environments/development.rb` 和 `production.rb` 加入寄信時預設的網站網址

```ruby
config.action_mailer.default_url_options = { :host => 'localhost:3000' }
```

* 在 view 中 加入可以顯示 flash 的 code

```ruby
<%if flash[:notice].present?%>
	<div class="alert alert-success text-center" role="notice">
		<%= flash[:notice] %>
	</div>
<%end%>

<%if flash[:alert].present?%>
    <div class="alert alert-danger text-center" role="alert">
    	<%= flash[:alert] %>
    </div>
<%end%>
```

* 預設不會產生 devise 的 view，因此必須輸入以下指令，就可以去更改 view 的樣式。

```ruby
rails g devise:views

#預設在 app/views/devise 
```

###產生 model

```ruby
rails generate devise User

#invoke  active_record
#create  db/migrate/20160117113653_add_devise_to_users.rb
#insert  app/models/user.rb
#route   devise_for :users
```

models/user

```ruby
class User < ActiveRecord::Base
  rolify
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
end
```


裡面有些是預設沒打開的，像是寄認證信之類的，需要再打開設定就可以了，migration 也是。  
也可以自行在 model 增加欄位，因為預設的可能是比較基本的。  

###驗證

在需要登入的 controller 加上

```ruby
before_action :authenticate_user!
```

在 view 中加入

```ruby
<% if current_user %>
	<%= link_to('登出', destroy_user_session_path, :method => :delete) %> 
	|
	<%= link_to('修改密碼', edit_registration_path(:user)) %>
<% else %>
	<%= link_to('註冊', new_registration_path(:user)) %> 
	|
	<%= link_to('登入', new_session_path(:user)) %>
<% end %>
```

編輯 `application_controller.rb` 

```ruby
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:username])
  end
end
```

### 串接社群登入 Authentication: 使用 Omniauth

[omniauth-facebook](https://github.com/mkdynamic/omniauth-facebook)  
[omniauth-google-oauth2](http://www.rubydoc.info/gems/omniauth-google-oauth2/0.2.5/frames)  
[omniauth-yahoo](https://github.com/timbreitkreutz/omniauth-yahoo)   
[omniauth-github](https://github.com/intridea/omniauth-github)  


#[Rolify](https://github.com/RolifyCommunity/rolify)
產生權限的 modle 和設定檔   

```ruby
rails generate rolify Role User

#invoke  active_record
#create  app/models/role.rb
#insert  app/models/role.rb
#create  db/migrate/20160117114226_rolify_create_roles.rb
#insert  app/models/user.rb
#create  config/initializers/rolify.rb
```

新增權限

```ruby
user = User.find(1)
user.add_role :admin
```
移除權限

```ruby
remove_role :admin
```
判斷權限

```ruby
user.has_role? :admin
#=> true
```

找擁有權限的人

```ruby
User.with_role(:admin)
#找單一權限

User.with_any_role(:user, :admin)
#找 a or b 權限

User.with_all_roles(:user, :admin)
#找 a + b 權限
```

### 對 resource 設定角色權限

可以直接指定這個權限可以進入哪些 resource

先在要設定的 model 裡  

>In the resource models you want to apply roles on, just add resourcify method. For example, on this ActiveRecord class:

```ruby
class Forum < ActiveRecord::Base
  resourcify
end
```
這樣就可以設定自己才可以看到自己的資料

```ruby
user.add_role :moderator, Forum.first 
#設定為第一個 Forum 的 resource

user.has_role? :moderator, Forum.first
#=> true

user.has_role? :moderator, Forum.last
#=> false

user.add_role :moderator, Forum 
#設定為所有的 Forum 的 resource
```

### resource 角色權限查詢

Instance level  

```ruby
forum = Forum.first
forum.roles
# => [ list of roles that are only binded to forum instance ]
forum.applied_roles
# => [ list of roles binded to forum instance and to the Forum class ]
```
Class level

```ruby
Forum.with_role(:admin)
# => [ list of Forum instances that has role "admin" binded to it ]
Forum.with_role(:admin, current_user)
# => [ list of Forum instances that has role "admin" binded to it and belongs to current_user roles ]
Forum.with_roles([:admin, :user], current_user)
# => [ list of Forum instances that has role "admin" or "user" binded to it and belongs to current_user roles ]

User.with_any_role(:user, :admin)
# => [ list of User instances that has role "admin" or "user" binded to it ]
User.with_role(:site_admin, current_site)
# => [ list of User instances that have a scoped role of "site_admin" to a site instance ]
User.with_role(:site_admin, :any)
# => [ list of User instances that have a scoped role of "site_admin" for any site instances ]
User.with_all_roles(:site_admin, :admin)
# => [ list of User instances that have a role of "site_admin" and a role of "admin" binded to it ]

Forum.find_roles
# => [ list of roles that binded to any Forum instance or to the Forum class ]
Forum.find_roles(:admin)
# => [ list of roles that binded to any Forum instance or to the Forum class with "admin" as a role name ]
Forum.find_roles(:admin, current_user)
# => [ list of roles that binded to any Forum instance or to the Forum class with "admin" as a role name and belongs to current_user roles ]
```


### Callbacks

```ruby
class User < ActiveRecord::Base
  rolify :before_add => :before_add_method

  def before_add_method(role)
    # do something before it gets added
  end
end

#四種 callbacks
#before_add
#after_add
#before_remove
#after_remove
```


#[Cancancan](https://github.com/CanCanCommunity/cancancan)
```ruby
rails generate cancan:ability
#create  app/models/ability.rb
```

ability.rb 主要是定義，每個角色擁有哪些權限，並在 view 或 controller ，設定條件時，就會到這邊去查看，是否符合此條件。

```ruby
#app/models/ability.rb

class Ability
  include CanCan::Ability

  def initialize(user) #這個 user 其實就是 devise 提供的 current_user
    if user.blank? # not logged in
      can [:new, :create], Forum #可以執行 Form Controller 裡的 new 和 create action
      cannot [:new], Comment  #無法執行 Comment Controller 裡的 new action
      basic_read_only #呼叫基本權限設定 Medthod
    elsif user.has_role?(:admin) #如果 role 為 admin
      can :manage, :all #可管理所有資源
    end
  end
  
  protected
     
  def basic_read_only
    can :read, Forum
  end
end

#:manage: 是指這個 controller 內所有的 action
#:read :  指 :index 和 :show
#:update: 指 :edit  和 :update
#:destroy:指 :destroy
#:create: 指 :new   和 :crate

```
也可以這樣寫

```ruby
can :read, [ Post, Comment ]
can [ :create, :update ], [ Post, Comment ]
```

自訂 Alias action
 
```ruby
alias_action :update, :destroy, :to => :modify
can :modify, Comment
```

自訂 method

```ruby
protected
     
  def basic_read_only
    can :read, Forum
  end
```

設定只能管理自己的post

```ruby
can :update, Post do |post|
    (post.user_id == user.id)
end
  
can :destroy, Post do |post|
    (post.user_id == user.id)
end

# 也可以這樣寫
can :update,  Post, user_id: user.id
can :destroy, Post, user_id: user.id
```

###[Authorizing controller actions](https://github.com/CanCanCommunity/cancancan/wiki/Authorizing-controller-actions)

load_and_authorized_resource

```ruby
class ProductsController < ActionController::Base
  load_and_authorize_resource
  def discontinue
    # Automatically does the following:
    # @product = Product.find(params[:id])
    # authorize! :discontinue, @product
  end
end

#指定 action
#load_and_authorize_resource :only => [:index, :show]
```


這個指令做了兩件事情

* load_resource

主要是可以自動加入 @instance ， 預設跟 Class 名稱相同，Article => @article

```ruby
def ArticlesController < ApplicationController
  load_resource
  
  def new
  end
  
  def show
    # @article automatically set to Article.find(params[:id])
  end
end
```

等於

```ruby
def ArticlesController < ApplicationController
  def new
    @article = Article.new
  end 

  def show
    @article = Article.find(params[:id])   
  end
end
```

* authorize_resource

代表將這個 Controller 加入權限的控制，並去 `model/ability.rb` 裡面判斷權限是否有效?  

```ruby
authorize_resource
```

也可以針對每個 action 去做設定

```ruby
def show
  @project = Project.find(params[:project])
  authorize! :show, @project
end
```

cancancan預設 ＠instance 變數 與 Controller Name 相同  
若＠instance變數 不為 Controller Name，
設定控管時須加入 "變數名稱"

```ruby
class TopicController < ApplicationController
 authorize_resource :post #變數名稱
 
 def index
  @post = Topic.all 
  #預設為 @topic
 end
 
end

#model/ability.rb 裡面要這樣下
can :read,   Post
can :create, Post
can :update, Post
```

* [check_authorization](https://github.com/CanCanCommunity/cancancan/wiki/Ensure-Authorization)

確保controller 都有加入授權管理

```ruby
class ApplicationController < ActionController::Base
  check_authorization
end
```

### if can?

會根據 `model/ability.rb` 去判斷是否有權限，並顯示。

```ruby
<% if can? :create, Project %>
  <%= link_to "New Project", new_project_path %>
<% end %>
```

###例外處理[Exception-Handling](https://github.com/CanCanCommunity/cancancan/wiki/Exception-Handling)

若 user 無權限進入， cancancan 會噴出一個 CanCan::AccessDenied exception。  
但那樣子比較不好看，所以我們要另外重新導向一個顯示無權限的頁面。

>`authorize_resource` or `authorize!` 才會丟例外。

```ruby
#application_controller.rb
rescue_from CanCan::AccessDenied do |exception|
  redirect_to root_url, :alert => exception.messag #導向另一個頁面
end

#render json: "Authorization failed. 權限錯誤，請洽管理人員。
#render :file => "#{Rails.root}/public/403.html", :status => 403, :layout => false
#respond_to do |format|
  #format.json { render nothing: true, status: :forbidden }
  #format.html { redirect_to main_app.root_url, :alert => exception.message }
#end
```

gem：  
[Device](https://github.com/plataformatec/devise)  
[Cancancan](https://github.com/CanCanCommunity/cancancan)  
[Rolify](https://github.com/RolifyCommunity/rolify)  
[Devise CanCanCan rolify Tutorial](https://github.com/RolifyCommunity/rolify/wiki/Devise---CanCanCan---rolify-Tutorial)

參考文件：  
[Devise + Rolify + Cancan](http://disco26.logdown.com/posts/210475-devise-rolify-cancan)  
[使用Devise＋Rolify + Cancan 控管權限](http://deveede.logdown.com/posts/206943-use-deviserolify-cancan-control-permissions)  
[Rails - Devise + Cancancan + Rolify](http://blog.jex.tw/blog/2015/04/12/rails-user/)  
[Cancan 實作角色權限設計的最佳實踐(1)](http://blog.xdite.net/posts/2012/07/30/cancan-rule-engine-authorization-based-library-1/)  
[Cancan 實作角色權限設計的最佳實踐(2)](http://blog.xdite.net/posts/2012/07/30/cancan-rule-engine-authorization-based-library-2/)  
[Cancan 實作角色權限設計的最佳實踐(3)](http://blog.xdite.net/posts/2012/07/30/cancan-rule-engine-authorization-based-library-3/)  
[使用者認證](https://ihower.tw/rails4/auth.html)