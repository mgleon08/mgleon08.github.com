---
layout: post
title: "Linux 指令"
date: 2018-04-09 17:03:31 +0800
comments: true
categories: linux
---

大部分人都只知道，基本指令 `ls` `cd` 等等，但其實還有很多好用的指令

<!-- more --> 

# screen

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

# top

> 能夠即時顯示系統中各個進程的資源佔用狀況


* [每天一個linux命令（44）：top命令](http://www.cnblogs.com/peida/archive/2012/12/24/2831353.html)
* [htop 觀測系統的狀態 - 取代 top 指令](http://blog.xuite.net/tolarku/blog/66114556-htop+%E8%A7%80%E6%B8%AC%E7%B3%BB%E7%B5%B1%E7%9A%84%E7%8B%80%E6%85%8B+-+%E5%8F%96%E4%BB%A3+top+%E6%8C%87%E4%BB%A4)
* [為什麼 Linux 的 htop 命令完勝 top 命令](https://linux.cn/article-3141-1.html)
* [Linux 用 ps 與 top 指令找出最耗費 CPU 與記憶體資源的程式](https://blog.gtwang.org/linux/ps-top-find-processes-by-cpu-memory-usage/)

# ps

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

# df

> 用來檢查linux伺服器的檔案系統的磁碟空間佔用情況。

```ruby
# -h 方便閱讀方式顯示
df -h
```

* [每天一個linux命令（33）：df 命令](http://www.cnblogs.com/peida/archive/2012/12/07/2806483.html)

# jobs

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

# curl

能夠通過http、ftp等方式下載文件，也能夠上傳文件。

```ruby
curl -O https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-64bit-static.tar.xz
```

* [一天一條Linux指令-curl](https://blog.csdn.net/u012313689/article/details/53038398)


# tar

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


# readlink

用來找出符號鏈接所指向的位置，如果是 symbolic link 加上 `-f` 可以一直查下去直到最後一個

```ruby
readlink -f /usr/bin/awk
```

* [readlink命令](https://blog.csdn.net/diabloneo/article/details/7173438)

# ln

ln 建立的連結分為 “硬連結” (hard link) 及 “軟連結” (symbolic link), 預設 ln 會使用 hard link。

```ruby
# -h hard link
# -s symbolic link
ln -s /home/wordpress /var/www/wpmu
```

* [ln — 建立連結指令– Linux 技術手札](https://caloskao.org/linux-use-ln-command-to-link-files-or-folders/)
* [ln — 建立連結指令](https://www.phpini.com/linux/ln-create-link-command)


# awk

文本分析工具，它是 Linux 中功能強大的數據處理引擎之一。相對於 grep 的查找，sed 的編輯

```ruby
awk ' /MA/ { print $1 }' list # 打印包含 MA 的行中的第一個單詞。
awk [options] 'script' file
```

* [每天學習一個命令：awk ](http://einverne.github.io/post/2018/01/awk.html)

# /dev/null

`/dev/null` 在 Unix 或 Linux 就像黑洞, 會將任何導入的東西吃掉, 簡單來說就是程式會照常執行, 但不會輸出任何執行結果

所以當要清空 file 內容時也可以執行

```ruby
cat /dev/null > access.log # 將空的內容丟給 access.log
cp /dev/null access.log # 複製一份空的蓋掉原本的檔案
``` 

* [5 Ways to Empty or Delete a Large File Content in Linux](https://www.tecmint.com/empty-delete-file-content-linux/)

# 2>&1

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

# scp

複製本機檔案到遠端機器，也可以反向

```ruby
scp [帳號@來源主機]:來源檔案 [帳號@目的主機]:目的檔案

# 從本地端複製到遠端
scp /path/file1 myuser@192.168.0.1:/path/file2

# 從遠端複製到本地端
scp myuser@192.168.0.1:/path/file2 /path/file1
```

* [Linux 的 scp 指令用法教學與範例：遠端加密複製檔案與目錄](https://blog.gtwang.org/linux/linux-scp-command-tutorial-examples/)

# xargs

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
