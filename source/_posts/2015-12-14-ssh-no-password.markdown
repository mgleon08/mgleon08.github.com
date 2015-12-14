---
layout: post
title: "遠端 SSH 免密碼登入(key)"
date: 2015-12-14 19:12:27 +0800
comments: true
categories: server 設定
---

用 ssh 連線到遠端 server 的時候，一般都要輸入帳號密碼來登入，但這會有幾點缺點

1. 輸入帳號密碼的同時，也會增加帳密被竊取的可能  
2. 當登入的頻率很高的時候，或是有很多台機器要登入的時候，就會覺得相當煩！！  

因此可以透過公開金鑰（Public Key）和私密金鑰（Private Key）對應的方式，去做登入，這樣以後就不用輸入密碼拉~

<!-- more -->

###Step 1. 

```
ssh root@123.123.12.1
sudo adduser --disabled-password deploy
sudo su deploy 
``` 
首先會先連到遠端 server 開新的帳號  
（因為root帳號權限很大，我們不希望每個人都用到root權限，而且root帳號是固定的，不夠安全）

先開個新帳號 deploy，就會產生 `home/deploy`   

*	`--disabled-password` 讓 deploy 無法用密碼登入  
*	`su` 就是切換身份

###Step 2.

```
ssh-keygen -t rsa

#Enter file in which to save the key (/root/.ssh/id_rsa): (不輸入，直接按Enter)
#Enter passphrase (empty for no passphrase): (不輸入，直接按Enter)
#Enter same passphrase again: (不輸入，直接按Enter)
```
再換到 deploy 後產生出 `id_rsa（private key）` , `id_rsa.pub （public key）`

接著複製本機的 `~/.ssh/id_rsa.pub` 到 `/home/deploy/.ssh/authorized_keys`

>之後連線, 就會用本機的 `id_rsa（private key）` 與遠端電腦的 `authorized_keys(public key)` 做認證

* 可以先在 (本機) `cat ~/.ssh/id_rsa.pub` 將 '所有' 字串複製
* 再到 (遠端)新增 `vi /home/deploy/.ssh/authorized_keys` ，將字串貼上去後 `:wq` 離開

###Step 3.
```
 chmod 644 /home/deploy/.ssh/authorized_keys
 chown deploy:deploy /home/deploy/.ssh/authorized_keys
```
更改權限，讓 group 和 other 可以讀  
`chmod` 改變權限  
`chown` 改變檔案擁有者

```
644 權限
owner  = rw- = 4+2+0 = 6  
group  = r-- = 4+0+0 = 4  
others = r-- = 4+0+0 = 4
```

  
接著就可以在 (本機) 直接  ssh deploy@123.123.12.1 就連進去囉!!

權限指令參考：
[鳥哥的私房菜](http://linux.vbird.org/linux_basic/0210filepermission.php#chown)