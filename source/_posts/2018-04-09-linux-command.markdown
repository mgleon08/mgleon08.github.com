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

* [每天一個linux命令（41）：ps命令](http://www.cnblogs.com/peida/archive/2012/12/19/2824418.html)

# df

> 用來檢查linux伺服器的檔案系統的磁碟空間佔用情況。

```ruby
# -h 方便閱讀方式顯示
df -h
```

* [每天一個linux命令（33）：df 命令](http://www.cnblogs.com/peida/archive/2012/12/07/2806483.html)

參考文件:

* [使用 Screen 指令操控 UNIX/Linux 終端機的教學與範例](https://blog.gtwang.org/linux/screen-command-examples-to-manage-linux-terminals/)
* [每天一個linux命令目錄](http://www.cnblogs.com/peida/archive/2012/12/05/2803591.html)