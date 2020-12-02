---
layout: post
title: "Linux 指令"
date: 2018-04-09 17:03:31 +0800
comments: true
categories: linux other
---

大部分人都只知道，基本指令 `ls` `cd` 等等，但其實還有很多好用的指令

<!-- more -->

* [permission - sudo, su, chown, chmod](#permission)
* [screen](#screen)
* [top](#top)
* [ps](#ps)
* [df](#df)
* [du](#du)
* [find](#find)
* [jobs](#jobs)
* [curl](#curl)
* [tar](#tar)
* [readlink](#readlink)
* [ln](#ln)
* [awk](#awk)
* [/dev/null](#/dev/null)
* [2>&1](#2&1)
* [scp](#scp)
* [xargs](#xargs)
* [> & >>](#arrow)
* [envsubst](#envsubst)
* [getent](#getent)
* [nohub](#nohub)
* [grep](#grep)

# <span id="permission"> permission - sudo, su, chown, chmod </span>

* sudo - 暫時切到最高權限
* su - 切換身份
* chown - 更改檔案／資料夾的擁有者和群組

```ruby
sudo chown root:root test.txt
# -R Recursive 所有底下檔案
sudo chown -R root:root test
```

* chmod - 更改檔案權限

```ruby
chmod 644 test.txt
```


# <span id="screen"> screen </span>

> 可以讓一個終端機當成好幾個來使用

### 參數

```ruby
# 開新的視窗
screen

# 列出目前所有執行中的 screen 工作環境
screen -ls

# 重新連接執行中的 screen 工作環境
screen -r [pid.tty.host]

# 開啟紀錄功能，會產生log檔案 screenlog.0
screen -L
cat screenlog.0

# 將廢棄的 screen 工作環境清除
screen -wipe
```

### 指令

```ruby
# 卸離 screen 工作環境
Ctrl + A + D
```

* [使用 Screen 指令操控 UNIX/Linux 終端機的教學與範例](https://blog.gtwang.org/linux/screen-command-examples-to-manage-linux-terminals/)

# <span id="top"> top </span>

> 能夠即時顯示系統中各個進程的資源佔用狀況


* [每天一個linux命令（44）：top命令](http://www.cnblogs.com/peida/archive/2012/12/24/2831353.html)
* [htop 觀測系統的狀態 - 取代 top 指令](http://blog.xuite.net/tolarku/blog/66114556-htop+%E8%A7%80%E6%B8%AC%E7%B3%BB%E7%B5%B1%E7%9A%84%E7%8B%80%E6%85%8B+-+%E5%8F%96%E4%BB%A3+top+%E6%8C%87%E4%BB%A4)
* [為什麼 Linux 的 htop 命令完勝 top 命令](https://linux.cn/article-3141-1.html)
* [Linux 用 ps 與 top 指令找出最耗費 CPU 與記憶體資源的程式](https://blog.gtwang.org/linux/ps-top-find-processes-by-cpu-memory-usage/)

# <span id="ps"> ps </span>

> Process Status的縮寫。用來列出系統中當前運行的那些進程。
>
> ps命令列出的是當前那些進程的快照，就是執行ps命令的那個時刻的那些進程，如果想要動態的顯示進程信息，就可以使用top命令。

```ruby
# 顯示較詳細的資訊
ps au
# 顯示所有包含其他使用者的行程
ps auxs
```

* [每天一個linux命令（41）：ps命令](http://www.cnblogs.com/peida/archive/2012/12/19/2824418.html)

# <span id="df"> df </span>

> 用來檢查linux伺服器的檔案系統的磁碟空間佔用情況。

```ruby
# -h 方便閱讀方式顯示
df -h
```

* [每天一個linux命令（33）：df 命令](http://www.cnblogs.com/peida/archive/2012/12/07/2806483.html)

# <span id="df"> du </span>

> 顯示目錄或是檔案的大小，與df命令不同的是 du 是對文件和目錄磁盤使用的空間的查看。

```ruby
# 找目前目錄底下，檔案大小排序的前 5 個
du -sh * | sort -nr | head
```

* [每天一個linux命令（34）：du 命令](http://www.cnblogs.com/peida/archive/2012/12/10/2810755.html)
* [Linux系統中df與du命令查看分區大小不一致問題分析](https://hk.saowen.com/a/e37a86c7ef13d56aeae532ac5bdd6547898449d9c5f6ba35ab819ceea2383b18)
* [[科普] df 和 du 指令為何有時候顯示不同和如何解決](https://medium.com/@jackyu/%E7%A7%91%E6%99%AE-df-%E5%92%8C-du-%E6%8C%87%E4%BB%A4%E7%82%BA%E4%BD%95%E6%9C%89%E6%99%82%E5%80%99%E9%A1%AF%E7%A4%BA%E4%B8%8D%E5%90%8C%E5%92%8C%E5%A6%82%E4%BD%95%E8%A7%A3%E6%B1%BA-91ed73c15dbe)
* [Linux 的 sort 排序指令教學與常用範例整理](https://blog.gtwang.org/linux/linux-sort-command-tutorial-and-examples/)
* [linux如何找出佔用較大空間的檔案](https://www.dreamjtech.com/content/linux%E5%A6%82%E4%BD%95%E6%89%BE%E5%87%BA%E4%BD%94%E7%94%A8%E8%BC%83%E5%A4%A7%E7%A9%BA%E9%96%93%E7%9A%84%E6%AA%94%E6%A1%88)


# <span id="find"> find </span>

> 支援非常多的搜尋選項，可以依照權限、擁有者、群組、檔案類型、日期與大小等條件來搜尋

```ruby
# 刪除 css gz js 大於 30 天以上的檔案
find *.css *.gz *.js -mtime +30 | xargs rm -rf
```

> 一次刪除多個檔案

```ruby
find . -type f -iname 'filename*' -delete
```

* [Unix/Linux 的 find 指令使用教學、技巧與範例整理](https://blog.gtwang.org/linux/unix-linux-find-command-examples/)
* [Questions Tags Users Unanswered Jobs How to delete files with the same extension in one go using rm?](https://askubuntu.com/questions/721808/how-to-delete-files-with-the-same-extension-in-one-go-using-rm)

# <span id="jobs"> jobs </span>

> 可以看到背景目前有的指令狀況

```ruby
& # 指令後面加上 `&`，可將指令丟到後台中去執行
ctrl + z #前台任務丟到後台中暫停
jobs # 顯示目前所有的 jobs 狀態
[1]  + 93594 running    sleep 10
fg %jobnumber 將後台的任務拿到前台來處理
bg %jobnumber 將任務放到後台中去處理
```

參考文件:

* [使用 Screen 指令操控 UNIX/Linux 終端機的教學與範例](https://blog.gtwang.org/linux/screen-command-examples-to-manage-linux-terminals/)
* [每天一個linux命令目錄](http://www.cnblogs.com/peida/archive/2012/12/05/2803591.html)
* [man7 hier](http://man7.org/linux/man-pages/man7/hier.7.html)
* [ubuntu hier](http://manpages.ubuntu.com/manpages/xenial/man7/hier.7.html)
* [鳥哥的私房菜](http://linux.vbird.org/)

# <span id="curl"> curl </span>

能夠通過http、ftp等方式下載文件，也能夠上傳文件。

```ruby
curl -O https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-64bit-static.tar.xz

// -L 如果有做跳轉，-L 可以一直 follow 下去
```

* [一天一條Linux指令-curl](https://blog.csdn.net/u012313689/article/details/53038398)


# <span id="tar"> tar </span>

壓縮，解壓縮

* `x` 解壓縮
* `j` bzip2
* `z` bzip
* `J` xz
* `v` verbose
* `f` 後指定要解的檔案，或是壓縮後的指定檔名

```ruby
tar xf ffmpeg-release-64bit-static.tar.xz
```

下載 + 解壓縮

```ruby
curl -L https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-64bit-static.tar.xz | tar Jxvf - -C /usr/local/src/
```

linux ffmpeg 下載流程

先將原本的 `/usr/local/bin/ffmpeg` 移除

1. 下載 `curl -O https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-64bit-static.tar.xz`
2. 解壓縮 `tar xf ffmpeg-release-64bit-static.tar.xz`
3. 將 ffmpeg 移到 bin/ `mv ffmpeg-4.0.2-64bit-static/ffmpeg /usr/local/bin/`
4. 更改權限 `chmod +x /usr/local/bin/ffmpeg`

* [每天一個linux命令（28）：tar命令](http://www.cnblogs.com/peida/archive/2012/11/30/2795656.html)
* [GNU / Linux 各種壓縮與解壓縮指令](http://note.drx.tw/2008/04/command.html)


# <span id="readlink"> readlink </span>

用來找出符號鏈接所指向的位置，如果是 symbolic link 加上 `-f` 可以一直查下去直到最後一個

```ruby
readlink -f /usr/bin/awk
```

* [readlink命令](https://blog.csdn.net/diabloneo/article/details/7173438)

# <span id="ln"> ln </span>

ln 建立的連結分為 “硬連結” (hard link) 及 “軟連結” (symbolic link), 預設 ln 會使用 hard link。

```ruby
# -h hard link
# -s symbolic link
ln -s /home/wordpress /var/www/wpmu
```

```ruby
ln -s ~/test.txt link.txt
cat link.txt
# this is text.txt
```

* [ln — 建立連結指令– Linux 技術手札](https://caloskao.org/linux-use-ln-command-to-link-files-or-folders/)
* [ln — 建立連結指令](https://www.phpini.com/linux/ln-create-link-command)


# <span id="awk"> awk </span>

文本分析工具，它是 Linux 中功能強大的數據處理引擎之一。相對於 grep 的查找，sed 的編輯

```ruby
# awk 動作 文件名
awk '{print $0}' test.txt
# $0 印出所有的，$1, $2..，則是根據指的是第1個字段, 第2個字段..
awk -F '/' '/MA/ { print $1 }' test.txt
# 打印包含 MA 的行中的第一個單詞。
pwd | awk -F '/' '{print NR ") " $NF}'
# -F 指定分隔參數，NF 表示當前行有多少段落，$NF 最後一個字段
```
另外也提供了一些 func

```ruby
tolower()：轉小寫
length()：字串長度
substr()：子字串
rand()：隨機數
```


* [每天學習一個命令：awk ](http://einverne.github.io/post/2018/01/awk.html)
* [awk 入門課程](http://www.ruanyifeng.com/blog/2018/11/awk.html)

# <span id="/dev/null"> /dev/null </span>

`/dev/null` 在 Unix 或 Linux 就像黑洞, 會將任何導入的東西吃掉, 簡單來說就是程式會照常執行, 但不會輸出任何執行結果

所以當要清空 file 內容時也可以執行

```ruby
cat /dev/null > access.log # 將空的內容丟給 access.log
cp /dev/null access.log # 複製一份空的蓋掉原本的檔案
```

* [5 Ways to Empty or Delete a Large File Content in Linux](https://www.tecmint.com/empty-delete-file-content-linux/)

# <span id="2&1"> 2>&1 </span>

`>/dev/null 2>&1`

代表將左邊執行的結果丟給 `/dev/null`

而 `>` 是重新導向的意思，當左右兩邊沒有數字時，代表它會讀取左方程式的標準輸出 (也就是 fd=1) 重新導向給右邊的東西

因此 `2>&1` 代表將標準錯誤輸出導向給 `&1 (fd)`，而為什麼後面要加 `&` 因為如果沒有加的話系統會誤以為是檔案名稱，而不是 fd.

* 標準輸入    stdin  (fd 是 0)
* 標準輸出    stdout (fd 是 1)
* 標準錯誤輸出 stderr (fd 是 2)

> fd: file descriptor, 檔案描述子

參考文件

* [Unix 重新導向跟 2>&1](http://ibookmen.blogspot.com/2010/11/unix-2.html)

# <span id="scp"> scp </span>

複製本機檔案到遠端機器，也可以反向

```ruby
scp [帳號@來源主機]:來源檔案 [帳號@目的主機]:目的檔案

# 從本地端複製到遠端
scp /path/file1 myuser@192.168.0.1:/path/file2

# 從遠端複製到本地端
scp myuser@192.168.0.1:/path/file2 /path/file1
```

* [Linux 的 scp 指令用法教學與範例：遠端加密複製檔案與目錄](https://blog.gtwang.org/linux/linux-scp-command-tutorial-examples/)

# <span id="xargs"> xargs </span>

xargs 這個指令會標準輸入（standard input）讀取資料，並以空白字元或換行作為分隔，將輸入的資料切割成多個字串，並將這些字串當成指定指令（預設為 /bin/echo）執行時的參數。

```ruby
xargs

arg1 arg2
arg3

# arg1 arg2 arg3
```

```ruby
echo a b c d e f | xargs -n 3

# echo a b c
# echo d e f
```

* [Linux 系統 xargs 指令範例與教學](https://blog.gtwang.org/linux/xargs-command-examples-in-linux-unix/)
* [xargs 命令教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2019/08/xargs-tutorial.html)

# <span id="arrow"> > & >> </span>

```ruby
echo hi > test # 覆蓋整個檔案
echo hello >> test # 夾在檔案最後一行
```

# <span id="envsubst"> envsubst </span>

> The envsubst program substitutes the values of environment variables.

利用環境變數(environment variables)搭配 envsubst 來產生設定檔。

```ruby
# myconfig
My name is ${USER}
My home path is ${HOME}
```

```ruby
envsubst < myconfig.conf
# My name is foo
# My home path is /home/foo
```

重新導向產生設定檔案

```ruby
envsubst < myconfig.conf > config.conf
```

如果只想替換某個變數

```ruby
envsubst '$USER' < myconfig.conf > config.conf
```

* [利用 Linux 指令 envsubst 產生設定檔](https://myapollo.com.tw/2018/02/23/linux-command-envsubst/)

# <span id="getent"> getent </span>

查看系統的數據庫中的相關記錄

```ruby
getent group
```
* [getent命令](https://ywnz.com/linux/getent/)

# <span id="nohub"> nohub </span>

讓程式可以在離線或登出系統後繼續執行

開頭 `nohup` 結尾 `&`

```ruby
# 讓程式登出後可繼續執行
nohup /path/my_program &

# 執行後會產生
nohup.out

# 可以使用 tail 看最新的輸出
tail -f nohup.out
```

* [nohub](https://blog.gtwang.org/linux/linux-nohup-command-tutorial/)

# <span id="grep"> grep </span>

find all files

```ruby
grep -rnw '/path/to/somewhere/' -e 'pattern'

# -r or -R is recursive,
# -n is line number, and
# -w stands for match the whole word.
# -l (lower-case L) can be added to just give the file name of matching files
```

* [How do I find all files containing specific text on Linux?](https://stackoverflow.com/questions/16956810/how-do-i-find-all-files-containing-specific-text-on-linux)

# <span id="netstat"> netstat </span>

列出所有 listening

```ruby
netstat -l

# 包含 non listening
netstat -a
```

列出 TCP, UDP

```ruby
# TCP
netstat -at

# UDP
netstat -au
```

列出統計數據

```ruby
netstat -s
```

查出程式

```ruby
# 使用 TCP 的程式
netstat -pt
netstat -ap | grep xxx
```

不要解析 DNS

```ruby
netstat -an
```

* [使用 Netstat 指令檢測網路的技巧](https://blog.gtwang.org/linux/linux-netstat-command-examples/)
