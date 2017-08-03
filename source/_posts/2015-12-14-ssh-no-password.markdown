---
layout: post
title: "遠端 SSH 免密碼登入(key) 設定"
date: 2015-12-14 19:12:27 +0800
comments: true
categories: server
---

用 ssh 連線到遠端 server 的時候，一般都要輸入帳號密碼來登入，但這會有幾點缺點

1. 輸入帳號密碼的同時，也會增加帳密被竊取的可能
2. 當登入的頻率很高的時候，或是有很多台機器要登入的時候，就會覺得相當煩！！

因此可以透過公開金鑰（Public Key）和私密金鑰（Private Key）對應的方式，去做登入，這樣以後就不用輸入密碼拉~

<!-- more -->

### Step 1.

```ruby
ssh root@123.123.12.1
sudo adduser --disabled-password deploy
sudo su deploy
```
首先會先連到遠端 server 開新的帳號
（因為root帳號權限很大，我們不希望每個人都用到root權限，而且root帳號是固定的，不夠安全）

先開個新帳號 deploy，就會產生 `home/deploy`

*	`--disabled-password` 讓 deploy 無法用密碼登入
*	`su` 就是切換身份

### Step 2. (也可以參考下面的 快速複製 SSH public key 到遠端主機)

```ruby
ssh-keygen -t rsa

#Enter file in which to save the key (/root/.ssh/id_rsa): (不輸入，直接按Enter)
#Enter passphrase (empty for no passphrase): (不輸入，直接按Enter)
#Enter same passphrase again: (不輸入，直接按Enter)
```
再（本機）輸入 `ssh-keygen -t rsa` 產生出 `id_rsa（private key）` 和 `id_rsa.pub （public key）`

接著複製（本機）的 `~/.ssh/id_rsa.pub` 到 `/home/deploy/.ssh/authorized_keys`(自己新增 .ssh 資料夾和 authorized_keys 檔案)

>之後連線, 就會用（本機）的 `id_rsa（private key）` 與遠端電腦的 `authorized_keys(public key)` 做認證

* 可以先在 (本機) `cat ~/.ssh/id_rsa.pub` 將 '所有' 字串複製
* 再到 (遠端)新增 `vi /home/deploy/.ssh/authorized_keys` ，將字串貼上去後 `:wq` 離開

### Step 3.

```ruby
 chmod 644 /home/deploy/.ssh/authorized_keys
 chown deploy:deploy /home/deploy/.ssh/authorized_keys
```
更改權限，讓 group 和 other 可以讀
`chmod` 改變權限
`chown` 改變檔案擁有者

```ruby
644 權限
owner  = rw- = 4+2+0 = 6
group  = r-- = 4+0+0 = 4
others = r-- = 4+0+0 = 4
```

接著就可以在 (本機) 直接  ssh deploy@123.123.12.1 就連進去囉!!


```ruby
-rw------- (600) -- 只有屬主有讀寫權限。
-rw-r--r-- (644) -- 只有屬主有讀寫權限；而屬組用戶和其他用戶只有讀權限。
-rwx------ (700) -- 只有屬主有讀、寫、執行權限。
-rwxr-xr-x (755) -- 屬主有讀、寫、執行權限；而屬組用戶和其他用戶只有讀、執行權限。
-rwx--x--x (711) -- 屬主有讀、寫、執行權限；而屬組用戶和其他用戶只有執行權限。
-rw-rw-rw- (666) -- 所有用戶都有文件讀、寫權限。這種做法不可取。
-rwxrwxrwx (777) -- 所有用戶都有讀、寫、執行權限。更不可取的做法。
 
以下是對目錄的兩個普通設定：

drwx------ (700) - 只有屬主可在目錄中讀、寫。
drwxr-xr-x (755) - 所有用戶可讀該目錄，但只有屬主才能改變目錄中的內容
```

#SSH config 設定檔
新增檔案 `config` 到 `~/.ssh/`

```ruby
#~/.ssh/config

#設定連線的資料方式等等，只要設定需要用到的即可
Host [自訂名稱]
  HostName 192.168.11.24 # IP or Domain name
  PreferredAuthentications publickey # 優先用金鑰認證
  PasswordAuthentication no # 停用密碼認證
  PubkeyAuthentication yes # 只允許金鑰認證
  IdentityFile ~/.ssh/id_rsa # 透過 local 的私鑰去做身份認證
  ForwardAgent yes # agent forwarding
Port 22 # 連線的 port
User deploy # 使用者名稱
```

接著就可以直接 `ssh [自訂名稱]` 就會登入 deploy 的帳號了。

* [如何用config管理多個網站的ssh key和如何不用每一組輸入ssh的Pass Phrase](http://blog.alantsai.net/2016/03/ssh-config-ssh-agent-passphrase-management.html#WizKMOutline_1457361341463918)

#快速複製 SSH public key 到遠端主機

###方法1

```
scp -i ~/.ssh/id_rsa.pub deploy@test.com:/home/deploy/.ssh/
```
###方法2

```
ssh-copy-id -i ~/.ssh/id_rsa.pub deploy@test.com:
```

* [每天一個Linux指令- scp指令(遠端檔案加密拷貝 工具)](http://jashliao.pixnet.net/blog/post/164556993-%E6%AF%8F%E5%A4%A9%E4%B8%80%E5%80%8Blinux%E6%8C%87%E4%BB%A4--scp%E6%8C%87%E4%BB%A4%28%E9%81%A0%E7%AB%AF%E6%AA%94%E6%A1%88%E5%8A%A0%E5%AF%86%E6%8B%B7%E8%B2%9D-)
* [SSH Public Key 快速複製到遠端主機](https://blog.longwin.com.tw/2011/03/ssh-public-key-copy-2011/)

# 測試 SSH 是否有連線成功

```
ssh -T git@github.com
```

# ssh-add

ssh-add命令是把專用密鑰添加到ssh-agent的高速緩存中。該命令位置在 `/usr/bin/ssh-add`

1. 把專用密鑰新增到 ssh-agent 的高速緩存中：`ssh-add ~/.ssh/id_rsa`
2. 從ssh-agent中刪除密鑰： `ssh-add -d ~/.ssh/id_xxx.pub`
3. 查看ssh-agent中的密鑰： `ssh-add -l`

* [ssh-add 指令](http://man.linuxde.net/ssh-add)
* [ssh-agent命令](http://man.linuxde.net/ssh-agent)
* [Mac 上 ssh-add 永久將私鑰新增到 Keychain](http://icodeyou.com/2016/01/17/ssh-add-mac/)

# SSH agent forwarding  

* [Using SSH agent forwarding](https://developer.github.com/guides/using-ssh-agent-forwarding/)
* [SSH agent forwarding 的應用](https://ihower.tw/blog/archives/7837)
* [SSH agent 轉發](http://wiki.jikexueyuan.com/project/github-developer-guides/using-ssh-agent.html)

# 本地測試
若是要本地自行測試的話，可以到 `/etc/hosts` 去設定網址對應 ip 位置，這樣之後連該網址就會連線到指定的 ip

```ruby
111.111.111.111 test.com
```

只網頁連線到 `test.com` 就會連到 ip `111.111.111.111` 的主機上

參考網站：

* [鳥哥的私房菜 - 檔案權限](http://linux.vbird.org/linux_basic/0210filepermission.php#chmod)
* [Ruby on Rails 實戰聖經 網站佈署](https://ihower.tw/rails/deployment.html)
* [ssh命令](http://man.linuxde.net/ssh)
* [Connecting to GitHub with SSH](https://help.github.com/articles/connecting-to-github-with-ssh/)
