---
layout: post
title: "Ruby on Rails 裝機趴 (only Mac)"
date: 2016-07-22 19:58:37 +0800
comments: true
categories: ruby rails other
---

2018-06-30 update!

最近剛好新來了一台電腦，所有東西都要重新安裝，就順手把需要的東西都紀錄了一下，以便之後可以快速的安裝起來!

<!-- more -->

### 目錄

* [iTerm2](#iterm2)
* [Homebrew](#homebrew)
* [Git](#git)
* [GitHub](#github)
* [zsh & oh-my-zsh](#zsh)
* [rvm or rbenv](#rvmorrbenv)
* [RubyGems](#rubygems)
* [Rails](#rails)
* [Bundle](#bundle)
* [NVM](#nvm)
* [NPM](#npm)
* [Yarn](#yarn)
* [Sublime Text](#sublime)
* [Visual Studio Code](#vscode)
* [Vim](#vim)
* [Linux](#linux)

# <span id="iterm2">iTerm2</span>

>Terminal 終端機

* [iTerm2](https://www.iterm2.com/)

### tmux
* [終端機必備的多工良伴：tmux](http://blog.chh.tw/posts/tmux-terminal-multiplexer/)

# <span id="homebrew">Homebrew</span>

>Mac OS X 上的套件管理程式

```ruby
#安裝
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

#homebrew檢查 
brew doctor

#已經裝了哪些套件
brew list

#homebrew更新
brew update

#搜尋套件
brew search

#查詢套件資訊
brew info

#module會裝到/usr/local/opt/xxx
#安裝xxx模組 
brew install xxx

#更新xxx模組 
brew upgrade xxx
```

* [Homebrew](http://brew.sh/)

推薦套件

[searchbrew](http://searchbrew.com/)

```ruby
#postgresql
brew install postgresql
gem install pg
brew services start postgresql
brew services stop postgresql
brew services restart postgresql
createdb db_name
psql db_name

#mysql
brew install mysql
gem install mysql2
mysql.server start

#redis
brew install redis

#背景啟動
redis-server --daemonize yes
redis-server &

#清除
redis-cli flushall

#imagemagick
brew install imagemagick
gem install rmagick

#ffmpeg
brew install ffmpeg

#MediaInfo
brew install mediainfo

#wget
brew install wget
```

* [How to start PostgreSQL server on Mac OS X?
](https://stackoverflow.com/questions/7975556/how-to-start-postgresql-server-on-mac-os-x)

# <span id="git">Git</span>

>版本控制

```ruby
#安裝 git
brew install git

#設定輸出顏色
git config --global color.ui true

#設定 git 的 user 和 email
git config --global user.name "YOUR NAME"
git config --global user.email "YOUR@EMAIL.com"

#空白對有些語言是有影響的(像是Ruby)，因此我們會希望 Git 去忽略空白的變化
git config --global apply.whitespace nowarn

#讓 git 記住你，不需每次上傳 code 都要打帳號密碼
git config --global credential.helper store 
```

### Git GUI 

* [SourceTree](https://www.sourcetreeapp.com/)

```ruby
#建立 command line 快捷
ln -s /Applications/SourceTree.app/Contents/Resources/stree /usr/local/bin/
```
 
* [GitX](http://gitx.frim.nl/)
* [tig](http://jonas.nitro.dk/tig/manual.html#view-scrolling)

### Git 指令
* [Git 指令操作手冊](http://mgleon08.github.io/blog/2015/12/27/git-command/)

# <span id="github">GitHub</span>

設定SSH連接

>若要沿用舊電腦的 key ， 將舊電腦的 id_rsa & public key 複製過來即可，但因為新電腦第一次登入，會要求輸入就電腦的 key 的密碼

```ruby
#本機產生 ssh key (id_rsa（private key）id_rsa.pub（public key）) ~/.ssh/
ssh-keygen -t rsa -C [email]

#設定權限(預設應該就有設好，若是自己新開檔案就要設定)
chmod 600 id_rsa
chmod 644 id_rsa.pub

#在 github 上新增新的 ssh
將 id_rsa.pub 內容複製過去
 
#測試是否能連上 github
ssh -T git@github.com

#查看 ssh 功能
man ssh

#將 key 加入 ssh-agent memory
ssh-add ~/.ssh/id_dsa

#檢查目前的 key
ssh-add -l

#刪除 ssh-agent key
ssh-add -d ~/.ssh/id_dsa.pub

#刪除所有 ssh-agent key
ssh-add -D ~/.ssh/id_dsa.pub

#將 key 永久紀錄
ssh-add -K [path/to/your/ssh-key]
```

* [Using SSH agent forwarding](https://developer.github.com/guides/using-ssh-agent-forwarding/)
* [SSH agent forwarding 的應用](https://ihower.tw/blog/archives/7837)  
* [ssh命令](http://man.linuxde.net/ssh)
* [鳥哥的私房菜 - 權限指令](http://linux.vbird.org/linux_basic/0210filepermission.php#chmod)


# <span id="zsh">zsh & oh-my-zsh</span>

>shell & zsh 的 framework 套件

```ruby
#安裝 zsh
brew install zsh

#將 shell 預設改成 zsh，改完記得重開 iterm or source ~/.zshrc
chsh -s /bin/zsh

#查看目前使用是哪個 shell
echo $SHELL

#下載 oh-my-zsh
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh

#將 oh-my-zsh 預設的設定複製到 ./.zshrc (.zshrc 需要自行產生)
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc

#安裝 zsh-completions
brew install zsh-completions

#加入以下兩行來啟動 zsh-completions
#zsh-completions
fpath=(/usr/local/share/zsh-completions $fpath)

#同時還需要 rebuild zsh 的 .zcompdump
rm -f ~/.zcompdump; compinit

#更改 theme ( 到 .zshrc 更改 ZSH_THEME 參數 )
ZSH_THEME="edvardm"

#啟動 oh-my-zsh 內建的套件，要看有哪些套件可以去 ~/.oh-my-zsh 裡面的 plugins 看裡面的設定( 到 .zshrc 更改 plugins 參數 )
plugins=(git ruby rbenv github gitignore rails rake python z)

```
* [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)
* [bash 轉移 zsh (oh-my-zsh) 設定心得](http://icarus4.logdown.com/posts/177661-from-bash-to-zsh-setup-tips)


# <span id="rvmorrbenv">rvm or rbenv</span>

>管理 ruby & gem 工具

### rvm
* [RVM and Gemsets Ruby版本控制](http://mgleon08.github.io/blog/2016/02/15/rvm-and-gemsets/)  
* [rvm 使用指南](https://ruby-china.org/wiki/rvm-guide)  

### rbenv

```ruby
#安裝
brew install rbenv ruby-build

#將指令放到 ~/.zshrc or ~/.bashrc
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"

#列出所有 ruby 版本
rbenv install --list

#列出安装的版本
rbenv versions               

#列出正在使用的版本
rbenv version    
            
#安裝 ruby (請依照當下最新的版本)
rbenv install 2.3.1

#設定預設版本
rbenv global 2.3.1

#確定目前使用的 ruby 是透過 rbenv 而不是內建的(記得重開視窗)
which ruby
#=> 確保有 .rbenv/shims/ruby

#project 加上 .ruby-version 檔案裡面寫 ruby 版本 才能加入控管
2.3.1

#rbenv 指令
Some useful rbenv commands are:
   commands    List all available rbenv commands
   local       Set or show the local application-specific Ruby version
   global      Set or show the global Ruby version
   shell       Set or show the shell-specific Ruby version
   install     Install a Ruby version using ruby-build
   uninstall   Uninstall a specific Ruby version
   rehash      Rehash rbenv shims (run this after installing executables)
   version     Show the current Ruby version and its origin
   versions    List all Ruby versions available to rbenv
   which       Display the full path to an executable
   whence      List all Ruby versions that contain the given executable
```

rbenv-gemset

```ruby
#安裝 rbenv-gemset
brew install rbenv-gemset

#rbenv gemset 指令
possible commands are:
  active
  create [version] [gemset]
  delete [version] [gemset]
  file
  init [gemset]
  list
  version
  
#使用方法
將想要使用的 gemset 名稱，放到 .rbenv-gemsets 檔案即可，在該專案執行 bundle 就會對設定好的 gemset 進行操作
若是沒有設定該檔案就會是 global 的 bundle
```

* [rbenv 使用指南](https://ruby-china.org/wiki/rbenv-guide)  
* [rbenv-gemset](https://github.com/jf/rbenv-gemset)
* [Setting up and installing rbenv, ruby-build, rubies, rbenv-gemset, and bundler](https://gist.github.com/MicahElliott/2407918)

# <span id="rubygems">RubyGems</span>

>RubyGems 是 Ruby 的套件管理系統，讓你輕易安裝及管理 Ruby 函式庫。

[RubyGems.org](https://rubygems.org/)

```ruby
#RubyGems 的版本
gem -v

#升級RubyGems的版本
gem update --system 

#安裝某個套件(加上 --no-ri --no-rdoc 可以不要產生預設的 RDoc和ri文件)
gem install gem_name --no-ri --no-rdoc

#列出安裝的套件
gem list 

#更新最新版本
gem update gem_name 

#更新所有你安裝的Gems
gem update 

#安裝特定版本
gem install -v x.x.x gemname

#反安裝
gem uninstall gem_name 

#移除所有 gem
gem uninstall -aIx

#移除舊版本的 gem
gem cleanup

#顯示要移除的有哪些
gem cleanup -d

#移除特定 gem 舊版本
gem cleanup rails
```

# <span id="rails">Rails</span>

```ruby
#後面的參數是不下載文件，可以省很多安裝時間
gem install rails --no-ri --no-rdoc
```

# <span id="bundle">Bundle</span>
> 管理應用程式 Gem 依存性(dependencies)管理工具，它會根據 Gemfile 的設定自動下載及安裝 Gem 套件

```ruby
gem install bundler

#Cleans up unused gems in your bundler directory
bundle clean [--force]
#Options:
#--force: forces clean even if --path is set
```

* [bundle](http://bundler.io/)

# <span id="nvm">NVM</span>

>管理 npm 工具，類似 rvm

```ruby
#根據當下最新版本
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.4/install.sh | bash

#將以下放到自己的 ~/.zshrc or ~/.bashrc or .bash_profile 下面（預設會自動放好，但還是去確定一下）
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm

#重新載入 Shell
. ~/.nvm/nvm.sh or source ~/.zshrc

# 安裝穩定版本的 NodeJS
nvm install stable 

#顯示目前可以安裝的版本
nvm ls-remote

#安裝 NodeJS
nvm install <version>

#安裝穩定版本的 NodeJS
nvm install stable 
 
#使用版本，只有在當下，重新開新tab就會消失
nvm use stable 
 
#設定預設版本，永久
nvm alias default stable

#看目前安裝所有版本
ls -a ~/.nvm/versions/node
```

```ruby
# package.json 設定
{ 
  "engines" : { 
    "node" : ">=0.10.3 <0.12" 
  } 
}
```

* [node版本管理工具nvm-Mac下安装及使用](https://segmentfault.com/a/1190000004404505)
* [在 Mac 上裝 NVM, RVM 等跑 Gulp, Grunt, Bower 環境](https://tenten.co/blog/install-gulp-grunt-bower-sass-susy-on-mac-with-nvm-rvm/)
* [Node.js 安裝與版本切換教學 (for MAC)](http://icarus4.logdown.com/posts/175092-nodejs-installation-guide)
* [package.json文件](http://javascript.ruanyifeng.com/nodejs/packagejson.html#toc6)
* [Node.js 環境設定-for mac](https://medium.com/@toumasaya/node-js-%E7%92%B0%E5%A2%83%E8%A8%AD%E5%AE%9A-for-mac-a2628836feaf)

# <span id="npm">NPM</span>

>套件管理

[npm](https://www.npmjs.com/)

資料夾一定會有 package.json

```ruby
#搜尋 npm 套件，但建議去網站上比較快
npm search

#本地安裝，會安裝在當前專案的 node_modules 目錄下
npm install <package name>

#全域安裝，會將套件安裝在統一的 npm 目錄底下
npm install -g <package name>

#列出專案使用套件
npm ls (-g 全域套件)

#更新專案套件
npm update (-g 全域套件)

#移除專案套件
npm uninstall <package name> (-g 全域套件)

#清快取
npm cache clean

#查詢 npm 儲存路徑
npm config get prefix

#自動安裝 package.json 套件定義檔中定義的所有套件
npm install

#安裝套件並儲存在 package.json 中
npm install <package name> --save #用於上線時必要的套件(react, bootstrap…)，會更新到package.json裡的Dependencies(上線依賴)
npm install <package name> --save-dev #用來安裝開發時用的工具(ex babel, webpack, webpack-dev-server…)，會更新到package.json裡的devDependencies(開發依賴)
```

### package.json

```ruby
#自動產生 package.json
npm init

{
  "name": "leon",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies":{#套件相依
  },
  "devDependencies": {#開發套件相依
    "lodash": "^4.15.0" #示範加上去的
  }
}
```

* [Npm 套件管理 & 常用開發工具介紹](http://www.slideshare.net/wantingj/npm-46801372)
* [Npm All cli](https://docs.npmjs.com/cli/access)

# <span id="yarn"> Yarn </span>

Facebook 開源的 Yarn，這是針對存儲在 npm 或 Bower 註冊表中的 JavaScript 模組的一個代理包管理器。

```ruby
brew install yarn

# -g 為 global 的意思，沒有加的話，會裝在當下的
yarn global add vue-cli

# 安裝指定套件 (會自動 save 到 package.json)
yarn add sass sass-loader node-sass

# 安裝 package.json 內其它套件
yarn install

# run project
yarn run start
```

* [yarn](https://yarnpkg.com/en/)
* [yarn github](https://github.com/yarnpkg/yarn)
* [yarn migrating from npm](https://yarnpkg.com/en/docs/migrating-from-npm)
* [[譯] Yarn 官方介紹: 一款新的 JavaScript 包管理器](https://github.com/cssmagic/blog/issues/67)
* [Facebook 新發佈的 Yarn JS 包管理器的 5 大功能](https://sheerdevelopment.com/posts/facebook-js-5)

# <span id="sublime">Sublime Text</span>

>編輯器

安裝好用的套件管理 [package control](https://packagecontrol.io/installation)

```ruby
All Autocomplete
AutoFileName
Alignment
Babel
Color Highlighter
CovertToUTF8
Emmet
GitGutter
SideBarEnhancement
BracketHighlighter
SublimeCodeIntel
BeautifyRuby
PrettyRuby
PrettyJson
PrettyYaml
AdvancedNewFile
View In Browser
Browser Refresh
JSHint

#theme
Spacegray
Material
```
* [Spacegray](https://github.com/kkga/spacegray)
* [Material](https://github.com/equinusocio/material-theme)

Prefrences > Setting-User

```ruby
{
	"bold_folder_labels": true,
	"caret_style": "phase",
	"color_scheme": "Packages/Theme - Spacegray/base16-eighties.dark.tmTheme",
	"fade_fold_buttons": false,
	"font_size": 20,
	"highlight_line": true,
	"ignored_packages":
	[
		"Vintage"
	],
	"line_padding_bottom": 1,
	"line_padding_top": 1,
	"spacegray_fileicons": true,
	"tab_size": 2,
	"theme": "Spacegray Eighties.sublime-theme",
	"translate_tabs_to_spaces": true,
	"trim_trailing_white_space_on_save": true,
	"word_wrap": true
}


#theme setting
"theme": "Spacegray Eighties.sublime-theme",
"spacegray_fileicons": true
```

建立快速鍵

```ruby
[
  { "keys": ["super+shift+k", ], "command": "pretty_ruby_format" },
  { "keys": ["super+shift+h"], "command": "htmlprettify" },
  { "keys": ["ctrl+super+k"],  "command": "beautify_ruby" }
]
```

設定快速command `subl .`
[OS X Command Line](https://www.sublimetext.com/docs/2/osx_command_line.html)

```ruby
#建立 bin 資料夾，建立軟連結到指定的檔案
mkdir ~/bin 
ln -s "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" ~/bin/subl

#若是 echo $PATH 沒有相對應的路徑，可以在 .zshrc 加上
export PATH="$HOME/bin:$PATH"
```

* [package control](https://packagecontrol.io/installation)
* [Sublime Text](https://www.sublimetext.com/)
* [Best Sublime Text 3 Themes of 2015 and 2016](https://scotch.io/bar-talk/best-sublime-text-3-themes-of-2015-and-2016)

#<span id="vscode"> Visual Studio Code </span>

### 套件安裝

找尋套件 `Command` + `Shift` + `P`

```js
// 在 terminal 直接開啟 vscode，一開始畫面就會問要不要設定了
code
```

### 推薦套件

```js
// Other
ESLint // 如果你有用程式碼規範的話
Prettier - code formatter // alt + shift + f
vscodeicon
CSScomb
Code Spell Checker
Bracket Pair Colorizer
// 其他可以看上面推薦的安裝
```

### Setting

```js
{
  "window.zoomLevel": 1, // 調整視窗的縮放比例。原始大小為 0
  "editor.fontSize": 12, // 字型大小
  "editor.renderControlCharacters": true, // 顯示控制字元
  "editor.renderWhitespace": "all", // 顯示空白字元
  "editor.tabSize": 2, // 預設縮排
  "editor.minimap.enabled": true, // 顯示 MiniMap
  "editor.minimap.renderCharacters": false, // MiniMap 不渲染實際字元
  "editor.formatOnSave": false, // 存擋時不進行排版
  "ruby.format": "rubocop",
  "[ruby]": {
    "editor.formatOnSave": true,
    "editor.tabSize": 2
  },
  "[go]": {
    "editor.tabSize": 4 // golang 縮排 4 格
  },
  "editor.renderIndentGuides": true, // 顯示縮排線
  "editor.wordWrap": "off", // 文字過長換行
  "files.trimTrailingWhitespace": true, // 檔案最後面留空格
  "breadcrumbs.enabled": true, // 顯示麵包屑
  "prettier.semi": true, // 結束是否加分號
  "prettier.singleQuote": true, // 單引號
  "prettier.trailingComma": "es5",  // 屬性後新增逗號
  "prettier.printWidth": 80, // 行寬
  "vetur.format.defaultFormatter.html": "prettyhtml",
  "explorer.openEditors.visible": 1, // 設定已開啟的頁面是否顯示於左側
  "go.gopath": "/Users/leon/go", // 設定 gopath
  // 接受的值: 'off'、'afterDelay、'onFocusChange' (編輯器失去焦點)
  // 、'onWindowChange' (視窗失去焦點)。若設為 'afterDelay'，可以在 "files.autoSaveDelay" 中設定延遲。
  "files.autoSave": "off",  // 控制已變更之檔案的自動儲存。
  "typescript.check.npmIsInstalled": false, // 檢查是否已安裝 NPM，以取得自動鍵入
  "ruby.intellisense": "rubyLocate",
  "ruby.locate": {
    "exclude": "{**/@(test|spec|tmp|.*),**/@(test|spec|tmp|.*)/**,**/*_spec.rb}",
    "include": "**/*.rb"
  },
  "cSpell.language": "en",  // 檢查拼字
  "cSpell.enabledLanguageIds": [
        "asciidoc",
        "c",
        "cpp",
        "csharp",
        "css",
        "go",
        "handlebars",
        "html",
        "jade",
        "javascript",
        "javascriptreact",
        "json",
        "latex",
        "less",
        "markdown",
        "php",
        "plaintext",
        "pub",
        "python",
        "restructuredtext",
        "ruby",
        "rust",
        "scss",
        "text",
        "typescript",
        "typescriptreact",
        "yml"
    ],
    "cSpell.userWords": [
        "elasticsearch",
        "kubectl",
        "kuma",
        "preload",
        "serializable",
        "serializer",
        "unscoped"
    ],
}
```

* [在 vscode 中統一 vue 編碼風格](http://www.52cik.com/2018/02/20/vscode-vue.html)
* [prettier.io options](https://prettier.io/docs/en/options.html)

### command line

```js
// 隱藏側邊欄
command + B
// 隱藏下邊欄
command + J
// Undo
command + Z
// Redo
command + shift + z
// New File
command + N
// New Window
command + shift + N
// Replace In Files
command + shift + H
// Replace
command + option + F
// Copy Line
option + shift + ↑↓
// Move Line
option + ↑↓
// Add cursor
command + option + ↑↓
// Comment One Line
command + K + C
// Remove Comment One Line
command + K + U
// Select All Occurrences of Find Match
command + shift + L
// open terminal
control + `
```

官方文件: 

* [visualstudio](https://code.visualstudio.com/)

# <span id="vim">Vim</span>

* <a href="{{ root_url }}/blog/2019/08/29/vim-basic/"> Vim Basic </a>
* [vimrc設定教學](http://wiki.csie.ncku.edu.tw/vim/vimrc)
* [vi指令說明(完整版)](http://www2.nsysu.edu.tw/csmlab/unix/vi_command.htm)

# <span id="linux">Linux 常用指令</span>

* [Linux常用Terminal命令與快捷鍵參考](http://jdev.tw/blog/3599/linux-terminal-commands-and-shortcuts)

參考文件： 
 
* [Ruby on Rails Mac 安裝教學](https://github.com/yuyueugene84/ntu_ror_training_course/blob/master/installation_mac.md)  
* [安裝 Rails 開發環境](https://ihower.tw/rails4/installation.html)
* [在 Mac 上裝 NVM, RVM 等跑 Gulp, Grunt, Bower 環境](https://tenten.co/blog/install-gulp-grunt-bower-sass-susy-on-mac-with-nvm-rvm/)
* [node版本管理工具nvm-Mac下安装及使用](https://segmentfault.com/a/1190000004404505)
* [為你自己學 Ruby on Rails 環境設定](http://railsbook.tw/chapters/02-environment-setup.html)
