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

#區域
1. 工作目錄 Working tree
2. 暫存準備遞交區 Staging Area (index索引)
3. 儲存庫 Repository

#狀態
1. 沒有被追蹤的檔案 Untracked files
2. 有修改、還沒準備要被遞交 Changes not staged for commit
3. 有修改、準備要被遞交的檔案(在Staging Area) Changes to be committed
4. 已經被遞交的檔案 Committed

#符號
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

#基本設定
* `git config --global user.name "<使用者名字>"`
* `git config --global user.email "<電子信箱>"`
* `git config --list`　看 Git 設定內容
* `git config --global alias.st status`　縮短字串（將 git status 縮寫為 git st）
* `git config --global color.ui true`　設定輸出顏色
* `git config --global apply.whitespace nowarn` 空白對有些語言是有影響的(像是Ruby)，因此我們會希望 Git 去忽略空白的變化

以上皆可直接 `vi .gitconfig` 去做修正

#基本功能

###basic
* `git --version` 查看目前版本。
* `git help [指令]` 查詢完整的文件。
* `git init` 建立 Repository。
* `git clone [url]` 複製遠端專案。
* `git mv [filename] [new filename]` 更改檔案名稱。(此更改檔案的動作會列入版本控制中)
* `git show [SHA1]` 查出該版本的相關資訊。(可搭配 `git log` 找出有問題的版本)

###add
* `git add .` 將當前目錄`所有`檔案加入到 Staging Area。(雖然方便但很容易會不小心加入一些其他不必要的檔案)
* `git add [filename]` 將檔案加入到 Staging Area。
* `git add -p`，可以選擇只加入檔案中修改的其中一部分。
* `git add -i`，會以互動方式詢問要加入到Staging Area裡的檔案。
* `git add -u`，只註冊已提交過的檔案到索引。

###rm
* `git rm [filename]` 刪除檔案，包括實體檔案。(此刪除檔案的動作會列入版本控制中)
* `git rm --cached a.txt` 只想刪除索引檔中的該檔，又要保留工作目錄下的實體檔案。

###commit
* `git commit` 將 Staging Area 檔案 commit。
* `git commit -m [commit 訊息]` 直接寫入 commit 訊息。
* `git commit -am [commit 訊息]` 等於 `git add .` + `git commit -m`。
* `git commit --amend` 合併到上一個 commit 訊息。

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

###checkout
* `git checkout [branch]` 切換到該 branch。
* `git checkout -b [branch]`建立一個新的 branch 並切換到該 branch。

>切換 working tree 的 branch 時,如果有檔案，在 staging area 或是 modified，會無法切換。可以先 commit，只要不 push 出去，再 `reset` 回來即可。或是用 `stash` 的方式

#合併
###merge
* `git merge [branch]` 將指定 branch 合併到目前的 branch
* `git merge [branch] --no-ff` 不使用 fast-forward 合併。可以保留分支合併的紀錄。

#修正

###reset

* `git reset` 將所有檔案從 Staging Area 到 Working tree。
* `git reset HEAD [filename]` 將檔案從 Staging Area 取消到 Working tree。
* `git reset --hard HEAD` 放棄目前所有修改，返回到最新的commit。
* `git reset HEAD^` 刪除最近一次的版本紀錄，但保留修改過的檔案。(回到working tree)
* `git reset --soft HEAD^` 刪除最近一次的版本紀錄，但保留修改過的檔案。(回到staging area)
* `git reset --hard HEAD^` 刪除最近一次的版本紀錄，清除修改過的檔案。(完全消失)

>HEAD 後面可接 `^` 表示回到上個版本， `^^` 回到上兩個版本，以此類推
>也可以用 `~`，或是 `^2` or `~2`，都不加就是回到目前最新的版本

* `git reset --hard ORIG_HEAD` 刪除最近一次的版本，但保留最後一次的變更。

###revert
>revert是新增一個commit來做還原，`git revert` 會用一個新的 commit 來回復所有的變更。(適合已經push出去給別人的情境)

###rebase
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
###cherry-pick
* `git cherry-pick` 手動挑出其他 branch 中的 commit，合併進來。
* `git cherry-pick [SHA1] -e` 建立版本前先編輯訊息。
* `git cherry-pick [SHA1] -n` 不建立版本，僅套用其變更。

###checkout
* `git checkout [filename]` 取消 Working tree 檔案。
* `git checkout master [filename]` 把 master 分支中最新版的檔案給還原。

#暫存
* `git stash` 會將所有已列入追蹤(tracked)的檔案建立暫存版。
* `git stash -u`　會包括所有已追蹤或未追蹤的檔案，全部都建立成暫存版。
* `git stash list` 顯示出所有的暫存清單。
* `git stash clear` 清除所有暫存。
* `git stash pop` 復原最新的操作並將他從暫存清單中移除。
* `git stash apply` 復原最新的操作，與`pop`唯一差別則是取回該版本 (其實是執行合併動作) 後，該暫存版還會留在 stash 清單上。
* `git stash drop` 刪除最新的暫存操作。

>指定stash ID（如：stash@{1}），則可以復原特定的操作

#遠端
* `git remote add origin [url]` 註冊遠端儲存庫。
* `git clone [url]` 將遠端儲存庫複製到本地，並建立工作目錄與本地儲存庫。
* `git push -u origin master` 將本地儲存庫中目前分支的所有相關物件推送到遠端儲存庫中。
* `git fetch` 將遠端儲存庫的最新版下載回來，下載的內容包含完整的物件儲存庫(object storage)。 這個命令不包含「合併」分支的動作。
* `git pull -u origin master` 將遠端資料庫拉下來，並且合併，相當於 `git fetch` + `git merge`。

>加上 `-u` 後續就只要 `git push` 和 `git pull` 即可。


* `git push --force` 可將 github 上的 `commit` 全部覆蓋掉。


#差異
* `git diff` 比對「Working tree」與「Staging Area」之間的差異。
* `git diff commit` 比對「Working tree」與「指定 commit 物件裡的那個 tree 物件」之間的差異。
* `git diff HEAD` 比對「Working tree」與「HEAD」之間的差異。
* `git diff --cached`比對「Staging Area」與「HEAD」之間的差異。(可改成 --staged)
* `git diff commit1(SHA1) commit2(SHA1)` 比較兩個 commit 差異。
* `git diff HEAD^ HEAD` 比較 HEAD 和 HEAD^ 差異。

#記錄

###log
* `git log` 查詢歷史紀錄。
* `git log -10` 限定輸出最近幾筆紀錄。
* `git log --oneline --graph --all --decorate` 圖像化 log。

###reflog
* `git reflog` 可列印出所有「歷史紀錄」的版本變化。
* `git log -g` 顯示 reflog 的詳細版本記錄。
* `git reflog delete HEAD@{1}` 刪除特定幾個版本的歷史紀錄。

#標籤

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

#其他
* `.gitkeep` 空目錄不會被 commit，必要時在目錄裡放 `.gitkeep`。
* `vi .gitignore` 編輯不要 commit 的檔案 此檔案也要 commit，通常是比較敏感的檔案，像是密碼之類的。
* [.gitignore ⼤集合](https://github.com/github/gitignore)
* [移除 git 上敏感檔案](https://help.github.com/articles/remove-sensitive-data/)

```
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch 檔案名稱' --prune-empty --tag-name-filter cat -- --all
```
* `.gitignore` not working

[.gitignore not working](http://stackoverflow.com/questions/11451535/gitignore-not-working)

```ruby
git rm -r --cached .
git add .
git commit -m "fixed untracked files"
```
* 開乾淨的 branch
[How to create a new empty branch for a new project](http://stackoverflow.com/questions/13969050/how-to-create-a-new-empty-branch-for-a-new-project)
```ruby
git checkout --orphan <branchname>
```

#alias
自己設定的一些 alias

```
alias g='git'

alias grs='git reset'
alias gci='git commit'
alias gcim='git commit -m'
alias gcim!='git commit --amend -m'
alias gst='git status'
alias gpl='git pull'
alias gplo='git pull origin'
alias gm='git merge'
alias gmo='git merge origin'
alias gd='git diff'

#git add
alias ga='git add'
alias gaa='git add --all'
alias gap='git add --patch'

#branch
alias gb='git branch'
alias gbd='git branch -d'
alias gbr='git branch --remote'
alias gba='git branch -a'

#git push
alias gp='git push'
alias gpo='git push origin'
alias gcp='git cherry-pick'
alias gpd='git push --dry-run'
alias gpoat='git push origin --all && git push origin --tags'
compdef _git gpoat=git-push
alias gpu='git push upstream'
alias gpv='git push -v'

#git checkout
alias gco='git checkout'
alias gcob='git checkout -b'
alias gcom='git checkout master'

#git rebase
alias grb='git rebase'
alias grba='git rebase --abort'
alias grbc='git rebase --continue'
alias grbi='git rebase -i'
alias grbm='git rebase master'
alias grbs='git rebase --skip'

#git stash
alias gsta='git stash'
alias gstd='git stash drop'
alias gstl='git stash list'
alias gstp='git stash pop'
alias gsts='git stash show --text'

#git log
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
```

官方文件：
[Git Pro](http://git-scm.com/book/zh-tw/v1)

參考資料：
[Git 版本控制系統](https://ihower.tw/git/)
[30 天精通 Git 版本控管](https://github.com/doggy8088/Learn-Git-in-30-days)
[連猴子都能懂的git入門](https://backlogtool.com/git-guide/tw/reference/)
[好麻煩部落格](http://blog.gogojimmy.net/2012/01/17/how-to-use-git-1-git-basic/)

練習：
[codeschool](https://try.github.io/levels/1/challenges/1)
[Easy-Git-Tutorial](http://dylandy.github.io/Easy-Git-Tutorial/)
[learnGitBranching](http://pcottle.github.io/learnGitBranching/)

