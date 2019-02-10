---
layout: post
title: "Docker - Docker Volumn"
date: 2016-11-27 16:28:35 +0800
comments: true
categories: server docker
---

Docker Volume 是個介於主機與容器間的共享資料夾，可用來匯入或匯出容器內的資料，或者共享容器間的資料

<!-- more -->

* Docker image 由多個 read-only 的 file system 疊加而成一個 stack，這些組合稱為 Union File System。
* 啟動 container 的時候，會在 stack 上方新增一個 read-write layer，更動都在這邊，砍掉 container 這邊也就沒了
* Volume 是為了要解決 container 之間資料共享與資料保存而提出的
* Volume 就是目錄或是檔案，可以繞過 UFS 以正常的檔案或目錄的方式存在 host 本機上
* 啟動 Container 的時候產生 volume，如果 Volume 掛載的目標路徑已經有檔案存在於 Image 上面，則 Image 上面的檔案會 copy 到 volume 上，但不包括 host directory

![](https://docs.docker.com/storage/images/types-of-mounts-volume.png)

```ruby
# 建立 volume，沒指定名稱會給個亂數
docker volume create --name db-data

# 列出所有的 volume
docker volume ls

# 列出所有沒有被關聯到 container 的 volume 的 VOLUME NAME
docker volume ls -qf dangling=true

# 一次刪除所有 volume
docker volume rm $(docker volume ls -qf dangling=true)

# 查看 volume 的資訊
docker volume inspect volume_name

# 查看某一個 container 的 volume 狀況
docker inspect -f '{{.Mounts}}' 825005afab43
```

### 實戰範例 named volume 1

> 先 create 再使用的 volume 稱作 named volume

建立新的 volume

```ruby
docker volume create --name db-data
docker volume ls # 查看是否建立成功
```

使用 volume

```ruby
# 將剛剛建立的 volume 掛載到 container 裡面的 /db/data
# 確認一開始資料夾是空的
docker run --rm -v db-data:/db/data -it ubuntu:14.04 ls -l /db/data
# 在 container 建立新的檔案
docker run --rm -v db-data:/db/data -it ubuntu:14.04 touch /db/data/file
在查看一次是否有檔案
docker run --rm -v db-data:/db/data -it ubuntu:14.04 ls -l /db/data
```

以上都是個別開 container 去操作，因此可以發現每次新的 container 資料還是會存在這，因為都是掛載同樣的 volume

### 實戰範例 named volume 2

`named volume` 沒有指定 loacl 的路徑，檔案會去哪裡?

```ruby
docker run -it -v /volume-test ubuntu:14.04 /bin/bash
docker volume ls

# 發現多了一個，因為沒指定名稱所以就產生亂碼
DRIVER              VOLUME NAME
local               c449638e58fe0c03961ce48411f31995b080f8cd4b13e8d67787cf775b1aa20a
```

```ruby
# 尋找 container 的 loacl volume, 825005afab43 是 container id
docker inspect -f '{{.Mounts}}' 825005afab43
[{volume c449638e58fe0c03961ce48411f31995b080f8cd4b13e8d67787cf775b1aa20a /var/lib/docker/volumes/c449638e58fe0c03961ce48411f31995b080f8cd4b13e8d67787cf775b1aa20a/_data /vo-test local  true }]
```

透過上面指令就可以找到 local 的檔案，照理說可以 cd 到 `/var/lib/docker/volumes/c449638e58fe0c03961ce48411f31995b080f8cd4b13e8d67787cf775b1aa20a/_data /vo-test`，但是 mac 無法，有找到以下有人也在詢問

* [Host path of volume](https://forums.docker.com/t/host-path-of-volume/12277)

### 實戰範例 host volume 1

`Host Volume` 指定 host 的資料夾

```ruby
# 讓 local /app dir 連結到 container 裡面的 /app
docker run --rm  -v ~/app:/app --workdir /app node yarn init -y
```

在 container 新增新的 `package.json` 在 local 的 `~/app` 就會產生一樣的檔案

### 實戰範例 host volume 2

```ruby
cd ~
mkdir volume-test
# 測試發現即使一開始沒新增 volume-test，執行指令後也是會在 local 新增這個檔案 
docker run -it -v /Users/leon/volume-test:/volume-test ubuntu:14.04 /bin/bash
# 會發現裡面也會有一個資料夾 volume-test，不管在 docker 的 /volume-test 或是 local /Users/leon/volume-test，新增檔案，另一邊都會同步
exit
# 離開了 container，會發現 /Users/leon/volume-test 裡面的檔案還是保留在裡面
docker run -it -v /Users/leon/volume-test:/volume-test ubuntu:14.04 /bin/bash
# 即使在開新的 container 檔案還是一樣會在
```


### Example 3.

讓 Container 和 Container 之間的資料共享

```ruby
docker run -it -v /data --name=container1 ubuntu:14.04 /bin/bash
docker run -it --volumes-from container1 --name=container2 ubuntu:14.04 /bin/bash

# 舊版參數
# --volumes-from 參數指定 Container Name 為 Container1 的 Volume 資料和Container2 做共享
```

### Example 4.

Dockerfile 指定

```ruby
FROM ubuntu:14.04
VOLUME /data
```

docker-compose.yml 指定

```ruby
version: '3'
services:
  db:
    image: postgres
    ports:
      - "5432"
  backend:
    build:
      context: test-backend
      args:
        UID: ${UID:-1001}
    volumes:
      - ./test-backend:/usr/src/app
    ports:
      - "8080:8080"
    depends_on:
      - db
    user: rails
  frontend:
    build:
      context: test-frontend
      args:
        UID: ${UID:-1001}
    volumes:
      - ./test-frontend:/usr/src/app
    ports:
      - "3000:3000"
    user: frontend
```

### 查看 volume 實際存取路徑

```ruby
# 查看名稱
docker volume ls

# 查看路徑
docker volume inspect --format '{{ .Mountpoint }}' volumeName
```

參考文件

* [Use volumes](https://docs.docker.com/storage/volumes/)
* [docker volume 容器卷的那些事（一）](https://deepzz.com/post/the-docker-volumes-basic.html)
* [Docker volume 簡單用法](https://julianchu.net/2016/04/19-docker.html)
* [Docker 實戰系列（三）：使用 Volume 保存容器內的數據](https://larrylu.blog/using-volumn-to-persist-data-in-container-a3640cc92ce4)
* [Day17：使用 Docker Volume 的功能 (一)](https://ithelp.ithome.com.tw/articles/10192397)
* [Day18：使用 Docker Volume 的功能 (二)](https://ithelp.ithome.com.tw/articles/10192703)
* [Docker Volume 初步閱讀與學習紀錄](http://blog.maxkit.com.tw/2017/03/docker-volume.html)
* [部署 長文慎入：將 Rails 程序部署到 Docker 容器中](https://ruby-china.org/topics/32459)
* [2018 年的 Rails 應用 Docker Image 包裝範例](https://5xruby.tw/en/posts/rails-docker-image)
* [Host path of volume](https://forums.docker.com/t/host-path-of-volume/12277)
