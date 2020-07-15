---
layout: post
title: "Git 指令操作手冊"
date: 2015-12-27 20:56:35 +0800
comments: true
categories: git
---

Git 博大精深，必須花很多時間去學習，從做中學會更快
因此這篇主要用來記錄 Git 的指令和操作，增加記憶，也方便後續查詢。

<!-- more -->

# 區域

1. 工作目錄 Working tree
2. 暫存準備遞交區 Staging Area (index索引)
3. 儲存庫 Repository

# 狀態

1. 沒有被追蹤的檔案 Untracked files
2. 有修改、還沒準備要被遞交 Changes not staged for commit
3. 有修改、準備要被遞交的檔案(在Staging Area) Changes to be committed
4. 已經被遞交的檔案 Committed

# 符號

1. HEAD
	* 永遠會指向「工作目錄」中所設定的「分支」當中的「最新版」。
	* 所以當你在這個分支執行 git commit 後，這個 HEAD 符號參照也會更新成該分支最新版的那個 commit 物件。
2. ORIG_HEAD
	* 簡單來說就是 HEAD 這個 commit 物件的「前一版」，經常用來復原上一次的版本變更。
	* [HEAD and ORIG_HEAD in Git](http://stackoverflow.com/questions/964876/head-and-orig-head-in-git)
3. FETCH_HEAD
	* 使用遠端儲存庫時，可能會使用 git fetch 指令取回所有遠端儲存庫的物件。這個 FETCH_HEAD 符號參考則會記錄遠端儲存庫中每個分支的 HEAD (最新版) 的「絕對名稱」。
4. MERGE_HEAD
	* 當你執行合併工作時 (關於合併的議題會在日後的文章中會提到)，「合併來源｣的 commit 物件絕對名稱會被記錄在 MERGE_HEAD 這個符號參照中。

# 基本設定

* `git config --global user.name "<使用者名字>"` (個別專案可用 --local)
* `git config --global user.email "<電子信箱>"` (個別專案可用 --local)
* `git config --list`　看 Git 設定內容
* `git config --global alias.st status`　縮短字串（將 git status 縮寫為 git st）
* `git config --global color.ui true`　設定輸出顏色
* `git config --global apply.whitespace nowarn` 空白對有些語言是有影響的(像是Ruby)，因此我們會希望 Git 去忽略空白的變化
* `git config --global alias.co checkout` 設定 alias
* `git config --global pager.branch false` git branch 不開啟 edit mode
* `git config --global push.default simple` git branch 不開啟 edit mode

以上皆可直接 `vim ~/.gitconfig` 去做修正

# 基本功能

### basic

* `git --version` 查看目前版本。
* `git help [指令]` 查詢完整的文件。
* `git init` 建立 Repository。
* `git clone [url]` 複製遠端專案。
* `git mv [filename] [new filename]` 更改檔案名稱。(此更改檔案的動作會列入版本控制中)
* `git show [SHA1]` 查出該版本的相關資訊。(可搭配 `git log` 找出有問題的版本)

### add

* `git add .` 將當前目錄`所有`檔案加入到 Staging Area。(雖然方便但很容易會不小心加入一些其他不必要的檔案)
* `git add [filename]` 將檔案加入到 Staging Area。
* `git add -u`，只註冊已提交過的檔案到索引。
* `git add -i`，會以互動方式詢問要加入到Staging Area裡的檔案。
* `git add -p`，可以選擇只加入檔案中修改的其中一部分。

[Commit only part of a file in Git](http://stackoverflow.com/questions/1085162/commit-only-part-of-a-file-in-git)

```
`y` stage this hunk for the next commit
`n` do not stage this hunk for the next commit
`q` quit; do not stage this hunk or any of the remaining ones
`a` stage this hunk and all later hunks in the file
`d` do not stage this hunk or any of the later hunks in the file
`g` select a hunk to go to
`/` search for a hunk matching the given regex
`j` leave this hunk undecided, see next undecided hunk
`J` leave this hunk undecided, see next hunk
`k` leave this hunk undecided, see previous undecided hunk
`K` leave this hunk undecided, see previous hunk
`s` split the current hunk into smaller hunks
`e` manually edit the current hunk
`?` print help
```
### rm

* `git rm [filename]` 刪除檔案，包括實體檔案。(此刪除檔案的動作會列入版本控制中)
* `git rm --cached a.txt` 只想刪除索引檔中的該檔，又要保留工作目錄下的實體檔案。

### commit

* `git commit` 將 Staging Area 檔案 commit。
* `git commit -m [commit 訊息]` 直接寫入 commit 訊息。
* `git commit -am [commit 訊息]` 等於 `git add .` + `git commit -m`。
* `git commit --amend` 合併到上一個 commit 訊息。
* `git commit --amend --no-edit` 合併到上一個 commit 訊息，並且不需要更改訊息

###status
* `git status` 檢視檔案的狀態。
* `git status -s` 顯示精簡的狀態。
* `git status -b` 顯示分支的名稱。

#分支

###branch
* `git branch` 列出所有本地端的 branch。
* `git branch -r` 列出所有遠端的 branch。
* `git branch -a` 列出所有本地及遠端的 branch。
* `git branch [branch]` 建立一個新的 branch。
* `git branch -m [oldbranch] [newbranch]` 修改分支的名稱， `-M` 強制。
* `git branch -d [branchname]` 刪除 branch，未合併的無法刪除，必須改用 `-D` 強制刪除。
* `git branch --merged maste` lists branches merged into master
* `git branch --merged` lists branches merged into HEAD (i.e. tip of current branch)
* `git branch --no-merged` lists branches that have not been merged

###checkout
* `git checkout [branch]` 切換到該 branch。
* `git checkout -b [branch]`建立一個新的 branch 並切換到該 branch。

>切換 working tree 的 branch 時,如果有檔案，在 staging area 或是 modified，會無法切換。可以先 commit，只要不 push 出去，再 `reset` 回來即可。或是用 `stash` 的方式

# 合併

### merge

* `git merge [branch]` 將指定 branch 合併到目前的 branch
* `git merge [branch] --no-ff` 不使用 fast-forward 合併。可以保留分支合併的紀錄。

# 修正

### reset

* `git reset` 將所有檔案從 Staging Area 到 Working tree。
* `git reset HEAD [filename]` 將檔案從 Staging Area 取消到 Working tree。
* `git reset --hard HEAD` 放棄目前所有修改，返回到最新的commit。
* `git reset HEAD^` 刪除最近一次的版本紀錄，但保留修改過的檔案。(回到working tree)
* `git reset --soft HEAD^` 刪除最近一次的版本紀錄，但保留修改過的檔案。(回到staging area)
* `git reset --hard HEAD^` 刪除最近一次的版本紀錄，清除修改過的檔案。(完全消失)

>HEAD 後面可接 `^` 表示回到上個版本， `^^` 回到上兩個版本，以此類推
>也可以用 `~`，或是 `^2` or `~2`，都不加就是回到目前最新的版本

* `git reset --hard ORIG_HEAD` 刪除最近一次的版本，但保留最後一次的變更。

### revert

>revert是新增一個commit來做還原，`git revert` 會用一個新的 commit 來回復所有的變更。(適合已經push出去給別人的情境)

### rebase

* `git rebase` 重新修改特定分支的「基礎版本」，把另外一個分支的變更，當成這個分支的基礎。(如果接著直接 merge，則是會直接 fast-forward ，若想保留資訊則加上  `--no-ff`)
* [Git-rebase 小筆記](https://blog.yorkxin.org/posts/2011/07/29/git-rebase/)
* `git rebase -i [SHA1]` 可以只接更改某個 commit 之後所有的紀錄。
* [rebase -i 修正 commit 過的版本歷史紀錄1](https://github.com/doggy8088/Learn-Git-in-30-days/blob/master/docs/22%20%E4%BF%AE%E6%AD%A3%20commit%20%E9%81%8E%E7%9A%84%E7%89%88%E6%9C%AC%E6%AD%B7%E5%8F%B2%E7%B4%80%E9%8C%84%20Part%204%20(rebase).markdown)
* [rebase -i 修正 commit 過的版本歷史紀錄2](https://github.com/doggy8088/Learn-Git-in-30-days/blob/master/docs/23%20%E4%BF%AE%E6%AD%A3%20commit%20%E9%81%8E%E7%9A%84%E7%89%88%E6%9C%AC%E6%AD%B7%E5%8F%B2%E7%B4%80%E9%8C%84%20Part%205%20(rebase%202).markdown)
* `git rebase --onto <new base-commit> <current base-commit>` 指定要從哪裡開始接枝
* 有 conflict 修改好後，`git rebase --continue`
* 直接從 rebase 狀態離開 `git rebase --abort`

```
pick   = 要這條 commit ，什麼都不改
reword = 要這條 commit ，但要改 commit message
edit   = 要這條 commit，但要改 commit 的內容
squash = 要這條 commit，但要跟前面那條合併，並保留這條的 messages
fixup  = squash + 只使用前面那條 commit 的 message ，捨棄這條 message
exec   = 執行一條指令
```

### cherry-pick

* `git cherry-pick` 手動挑出其他 branch 中的 commit，合併進來。
* `git cherry-pick [SHA1] -e` 建立版本前先編輯訊息。
* `git cherry-pick [SHA1] -n` 不建立版本，僅套用其變更。

### checkout

* `git checkout [filename]` 取消 Working tree 檔案。
* `git checkout master [filename]` 把 master 分支中最新版的檔案給還原。

# 暫存

* `git stash` 會將所有已列入追蹤(tracked)的檔案建立暫存版。
* `git stash -u`　會包括所有已追蹤或未追蹤的檔案，全部都建立成暫存版。
* `git stash list` 顯示出所有的暫存清單。
* `git stash clear` 清除所有暫存。
* `git stash pop` 復原最新的操作並將他從暫存清單中移除。
* `git stash apply` 復原最新的操作，與`pop`唯一差別則是取回該版本 (其實是執行合併動作) 後，該暫存版還會留在 stash 清單上。(取出最新的一筆 stash 暫存資料. 但是 stash 資料不移除)
* `git stash drop` 刪除最新的暫存操作。
* `git stash show stash@{0}` 看暫存陣列裡的第一個

>指定stash ID（如：stash@{1}），則可以復原特定的操作

# 遠端

* `git remote add origin [url]` 註冊遠端儲存庫。
* `git clone [url]` 將遠端儲存庫複製到本地，並建立工作目錄與本地儲存庫。
* `git push -u origin master` 將本地儲存庫中目前分支的所有相關物件推送到遠端儲存庫中。
* `git fetch` 將遠端儲存庫的最新版下載回來，下載的內容包含完整的物件儲存庫(object storage)。 這個命令不包含「合併」分支的動作。
* `git pull -u origin master` 將遠端資料庫拉下來，並且合併，相當於 `git fetch` + `git merge`。

>加上 `-u` 後續就只要 `git push` 和 `git pull` 即可。
> `-u` 相當於 `--set-upstream` 用來追蹤遠端的分支，因此要取消追蹤為 `--unset-upstream`

* `git push --force` 可將 github 上的 `commit` 全部覆蓋掉。
* `git pull --rebase origin [brance]` pull 遠端用 rebase 的方式。

* [upstream](https://zlargon.gitbooks.io/git-tutorial/content/remote/upstream.html)
* [Git 更安全的強制推送，--force-with-lease](https://walterlv.github.io/post/safe-push-using-force-with-lease.html)

# 差異

* `git diff` 比對「Working tree」與「Staging Area」之間的差異。
* `git diff commit` 比對「Working tree」與「指定 commit 物件裡的那個 tree 物件」之間的差異。
* `git diff HEAD` 比對「Working tree」與「HEAD」之間的差異。
* `git diff --cached`比對「Staging Area」與「HEAD」之間的差異。(可改成 --staged)
* `git diff commit1(SHA1) commit2(SHA1)` 比較兩個 commit 差異。
* `git diff HEAD^ HEAD` 比較 HEAD 和 HEAD^ 差異。
* `git diff BRANCH` 比較當下 branch 和指定 branch 的差異。

# 記錄

### log

* `git log` 查詢歷史紀錄。
* `git log -10` 限定輸出最近幾筆紀錄。
* `git log --oneline --graph --all --decorate` 圖像化 log。
* `git log --oneline --author="leon"` 找出 leon 所有的 commit(`多個名稱 --author="leon1\|leon2"`)
* `git log --oneline --grep="rails"` 找出 commit 訊息有 rails 的字眼
* `git log -S "Ruby"` 找出 commit 的內容有包括 `Ruby` 的字
* `git log --oneline --since="9am" --until="12am"` 今天早上 9 點到 12 點之間所有的 Commit
* `git log --oneline --since="9am" --until="12am" --after="2017-01"`  2017 年 1 月之後，每天早上 9 點到 12 點的 Commit

### reflog
* `git reflog` 可列印出所有「歷史紀錄」的版本變化。
* `git log -g` 顯示 reflog 的詳細版本記錄。
* `git reflog delete HEAD@{1}` 刪除特定幾個版本的歷史紀錄。

[【狀況題】不小心使用 hard 模式 Reset 了某個 Commit，救得回來嗎？](https://gitbook.tw/chapters/using-git/restore-hard-reset-commit.html)

### rev-parse

`git rev-parse` 

可以把任意「參考名稱」或「相對名稱」解析出「絕對名稱」sha1

* `git rev-parse master` 看 master 的 sha1
* `git rev-parse HEAD` 看 HEAD 的 sha1
* `git rev-parse ORIG_HEAD`
* `git rev-parse HEAD^`
* `git rev-parse HEAD~5`
* `git rev-parse --abbrev-ref HEAD` 看 HEAD 的 sha1 對應的 branch 名稱 


# 標籤

* `git tag` 列出所有標籤。
* `git tag -n` 顯示標籤的列表和註解。
* `git tag [tagname]` 建立輕量標籤。
* `git tag [tagname] -d` 刪除輕量標籤。
* `git tag [tagname] -am [版本訊息]` 建立標示標籤

> 1. 輕量標籤 (lightweight tag)
	* 不可變更的暫時標籤
	* 可以添加名稱
> 2. 標示標籤 (annotated tag)
	* 可以添加打標簽者的名稱、email及日期
	* 可以添加名稱
	* 可以添加註解
	* 可以添加簽名

### push 標籤

* `git push origin v1.5` git push 並不會把標籤上傳到遠端，所以必須透過底下才行
* `git push origin --tags` 用 –tags 一次上傳上去

### 刪除標籤

* `git push origin :refs/tags/my_tag`
* `git tag -d <tagname></tagname>`

[[Git] 版本控制: 如何使用標籤(Tag)](https://blog.wu-boy.com/2010/11/git-%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%E6%A8%99%E7%B1%A4tag/)

# commit 順序

```
schema
refactor 現有的架構
new feture
refactor new feature
hotfix
```

# Blame

* `git blame [filename]` 可以看最後一個 commit 的人是誰
* `git blame -L [開始行數],[結束行數] [filename]`


# submodule
[Git Submodule 用法筆記](http://blog.chh.tw/posts/git-submodule/)


# git-bisect
利用 git 的 二分找法，來找出當初錯誤的版本是從哪個開始

```ruby
git bisect start #開始

git bisect good 4b274b160 #設定好的 commit SHA1
git bisect bad  079944e6  #設定壞的 commit SHA1

#接著就會從兩個 commit 中間的 commit 去問，是否這個 commit 是好的，可以回答
#git bisect good
#git bisect bad
#就會根據回答，再下去做二分法，直到找到，一開始壞的 commit 在哪

git bisect reset #結束
```


# git format-patch

git 製作成 patch 檔 然後 merge

```ruby
# 最新的 n 個 commit
git format-patch -n

# 清除之前的訊息
git am --abort

# 上patch
git am xxx.patch
git am *.patch

# 保存 diff
git diff file > mypatch.patch
git apply mypatch.patch
```

* [【Git】使用 format-patch 將 commit 打包成檔案 Use format-patch to carry commit](http://chris800731.blogspot.tw/2013/08/git-format-patch-commit-use-format.html)
* [5.3 分散式 Git - 專案的管理](https://git-scm.com/book/zh-tw/v1/%E5%88%86%E6%95%A3%E5%BC%8F-Git-%E5%B0%88%E6%A1%88%E7%9A%84%E7%AE%A1%E7%90%86)
* [Create patch or diff file from git repository and apply it to another different git repository](https://stackoverflow.com/questions/28192623/create-patch-or-diff-file-from-git-repository-and-apply-it-to-another-different)

# 其他

### gitkeep

* `.gitkeep` 空目錄不會被 commit，必要時在目錄裡放 `.gitkeep`。

### .gitignore

* `vi .gitignore` 編輯不要 commit 的檔案 此檔案也要 commit，通常是比較敏感的檔案，像是密碼之類的。

[.gitignore ⼤集合](https://github.com/github/gitignore)
[.gitignore not working](http://stackoverflow.com/questions/11451535/gitignore-not-working)

```ruby
git rm -r --cached .
git add .
git commit -m "fixed untracked files"
```

### 移除已經上 github 的敏感檔案

* [移除 git 上敏感檔案](https://help.github.com/articles/remove-sensitive-data/)

```ruby
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch 檔案名稱' --prune-empty --tag-name-filter cat -- --all

git filter-branch --tree-filter "rm -f config/database.yml"
# filter-branch 這個指令可以讓你根據不同的 filter，一個一個 Commit 的來處理它。
# --tree-filter 這個 filter，它可以讓你在 Checkout 到每個 Commit 的時候執行你指定的指令，執行完後再自動幫你重新再 Commit
```

### 開乾淨的 branch

[How to create a new empty branch for a new project](http://stackoverflow.com/questions/13969050/how-to-create-a-new-empty-branch-for-a-new-project)

```ruby
git checkout --orphan <branchname>
```

### 複製遠端的 branch 到本地新的 branch

1. 先 `git checkout` 到遠端的 branch，在遠端的 branch `git checkout branch`複製切到本地

```ruby
git checkout origin/development
git checkout -b development
```

2. 直接 fetch 遠端 branch 下來到新的 branch

```
git fetch origin <remote_branch_name>:<local_branch_name>
```

### 移除 untracked file

```ruby
#Show what will be deleted with the -n option
git clean -f -n
git clean -f
git clean -df #刪除 untracked directory
git clean -fX # 清除被忽略的檔案
```

[How do I remove local (untracked) files from my current Git branch?](http://stackoverflow.com/questions/61212/how-do-i-remove-local-untracked-files-from-my-current-git-branch)

### 針對每個節點刪除特定檔案

```ruby
git fliter-branch --tree--filter "rm -f config/password.txt"
```

### 移除local 遠端 branch

clear any remote-tracking branch which is no longer exist on the remote.

```ruby
git fetch --prune
```

### git 垃圾回收

```ruby
git gc
```

### 拯救已被刪除的 branch

```ruby
# 看目前 branch 的 sha1
git rev-parse <branch name>

# 或使用 git reflog 找找看有沒有之前的記錄
git reflog

# 建立新的 branch 並給予 sha1
git branch new_name <sha1>
```

### 拯救已被 hard reset 的節點

```ruby
# 查看 log 節點的 sha1
git reflog

# 強制回復到該節點
git reset <sha1> --hard
```

### 拯救已刪除的檔案

```ruby
# 會列出一堆檔案的 bash
git fsck --cache --unreachable
# 逐一檢視檔案內容即可救回失去的檔案 
git show <hash> 
```

 * [Git 救回已刪除的檔案](http://blog.hsatac.net/2012/07/git-restore-removed-files/)

### 取的 remote git 資訊

* [git-ls-remote](https://git-scm.com/docs/git-ls-remote.html)

```ruby
# 顯示特定遠端儲存庫的參照名稱。包含遠端分支與遠端標籤 -h 遠端 heads,  -t 顯示 tag 
git ls-remote .
```

### 找出 log 裡面的關鍵字

[How to search a Git repository by commit message?](https://stackoverflow.com/questions/7124914/how-to-search-a-git-repository-by-commit-message)

```ruby
# 所有 branch
git log --all --grep='Build 0051'

# 當前 branch
git log --grep='Build 0051'

# 找出所有 commit 內容裡的關鍵字
git grep 'Build 0051' $(git rev-list --all)
```

### 同時開多個工作目錄

[Git worktree: 同時開多個工作目錄](https://ihower.tw/blog/archives/8740?utm_campaign=CodeTengu&utm_medium=email&utm_source=CodeTengu_144)

```ruby
# 建立 linked working tree 
git worktree git worktree add -b hotfix ../hotfix master 

# 到該資料夾
cd ../hotfix

# 清除  linked working tree 
git worktree prune
```

### Reset 所有的 commit 的 author

[How to change the commit author for one specific commit?](https://stackoverflow.com/questions/3042437/how-to-change-the-commit-author-for-one-specific-commit)

```
git rebase -i --root
git commit --amend --reset-author --no-edit
grb --continue
```

### 找出某個 commit 什麼時候被 merge 和 branch 名稱

* [git-when-merged](https://github.com/mhagger/git-when-merged)

# alias

自己設定的一些 alias

可以建立在 `~/.gitconfig` 裡面，但是前面 `git` 就無法縮寫成 `g` 

```ruby
[alias]
  co = checkout
  br = branch
  st = status
  l  = log --oneline --graph
  ls = log --graph --pretty=format:"%h <%an> %ar %s"
```

也可以建立新的檔案 `.aliases`, 並在 `.zshrc` 加上 `source ~/.aliases`

```ruby
# base
alias g='git'

alias grs='git reset'
alias gst='git status'
alias gd='git diff'
alias gdca='git diff --cached'

# git add
alias ga='git add'
alias gaa='git add --all'
alias gap='git add --patch'

# git commit
alias gc='git commit'
alias gcm='git commit -m'
alias gcm!='git commit --amend -m'
alias gcam='git commit -a -m'

# git merge
alias gm='git merge'
alias gmo='git merge origin'

# branch
alias gb='git branch'
alias gbd='git branch -d'
alias gbr='git branch --remote'
alias gba='git branch -a'

# git push
alias gp='git push'
alias gpo='git push origin'
alias gcp='git cherry-pick'
alias gpd='git push --dry-run'
alias gpoat='git push origin --all && git push origin --tags'
alias gpu='git push upstream'
alias gpv='git push -v'

# git pull
alias gpl='git pull'
alias gplo='git pull origin'
alias gplrb='git pull --rebase'
alias gplrbo='git pull --rebase origin'

# git checkout
alias gco='git checkout'
alias gcob='git checkout -b'
alias gcom='git checkout master'

# git fetch origin
alias gf='git fetch'
alias gfo='git fetch origin'

# git rebase
alias grb='git rebase'
alias grba='git rebase --abort'
alias grbc='git rebase --continue'
alias grbi='git rebase -i'
alias grbm='git rebase master'
alias grbs='git rebase --skip'

# git stash
alias gsta='git stash save'
alias gstak='git stash save --keep-index'
alias gstau='git stash save --include-untracked'
alias gstaa='git stash apply'
alias gstd='git stash drop'
alias gstl='git stash list'
alias gstp='git stash pop'
alias gsts='git stash show'
alias gstc='git stash clear'
alias gsts='git stash show --text'

# git log
alias gl='git log'
alias glg='git log --stat --color'
alias glgp='git log --stat --color -p'
alias glgg='git log --graph --color'
alias glgga='git log --graph --decorate --all'
alias glgm='git log --graph --max-count=10'
alias glo='git log --oneline --decorate --color'
alias glol="git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
alias glola="git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --all"
alias glog='git log --oneline --decorate --color --graph'

# git peco
# interactive checkout
alias gcoi='git checkout $(git branch | cut -c 3- | peco)'
# interactive merge
alias gmi='git merge $(git branch | cut -c 3- | peco)'
# interactive add
alias gai='git add $(git status -s | cut -c 4- | peco)'

# cleanup
alias gbcup="git branch | egrep -v '(^\*|master|dev)'"
alias gbcup!="git branch | egrep -v '(^\*|master|dev)' | xargs -n 1 git branch -D"
alias gbrcup="git branch --remote | egrep -v '(^\*|origin/master|origin/dev)'"
alias gbrcup!="git branch --remote | egrep -v '(^\*|origin/master|origin/dev)' | xargs -n 1 git branch --remote -D"

alias gbmd="git branch --merged | egrep -v '(^\*|master|dev)'"
alias gbmd!="git branch --merged | egrep -v '(^\*|master|dev)' | xargs git branch -d"
alias gbrmd="git branch -r --merged | egrep -v '(^\*|origin/master|origin/dev)'"
alias gbrmd!="git branch -r --merged | egrep -v '(^\*|origin/master|origin/dev)' | xargs git branch -r -d"
```

官方文件：

* [Git Pro](http://git-scm.com/book/zh-tw/v1)

參考資料：

* [Git 版本控制系統](https://ihower.tw/git/)
* [30 天精通 Git 版本控管](https://github.com/doggy8088/Learn-Git-in-30-days)
* [連猴子都能懂的git入門](https://backlogtool.com/git-guide/tw/reference/)
* [好麻煩部落格](http://blog.gogojimmy.net/2012/01/17/how-to-use-git-1-git-basic/)
* [Commit only part of a file in Git](http://stackoverflow.com/questions/1085162/commit-only-part-of-a-file-in-git)
* [Git 風格指南](https://github.com/JuanitoFatas/git-style-guide)
* [How to Write a Git Commit Message](http://chris.beams.io/posts/git-commit/)
* [你知道 Git 是怎麼一回事嗎](http://kaochenlong.com/2017/08/11/git-on-modern-web/)
* [為你自己學 Git](https://gitbook.tw/)
* [Git 小說連載系列](https://kaochenlong.com/git-tips/)

練習：

* [codeschool](https://try.github.io/levels/1/challengesㄋ/1)
* [Easy-Git-Tutorial](http://dylandy.github.io/Easy-Git-Tutorial/)
* [learnGitBranching](http://pcottle.github.io/learnGitBranching/)

其他技巧：

* [使用 git rebase 避免無謂的 merge](https://ihower.tw/blog/archives/3843)
* [Git flow 開發流程](https://ihower.tw/blog/archives/5140)
* [A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)
* [git-flow 備忘清單](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_TW.html)
* [The magical (and not harmful) rebase](http://jeffkreeftmeijer.com/2010/the-magical-and-not-harmful-rebase/)
* [How to Write a Git Commit Message](http://chris.beams.io/posts/git-commit/#seven-rules)
* [A guide to using Github Pages](https://www.thinkful.com/learn/a-guide-to-using-github-pages/)
* [How can I delete all Git branches which have been merged?](https://stackoverflow.com/questions/6127328/how-can-i-delete-all-git-branches-which-have-been-merged)
