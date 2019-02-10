---
layout: post
title: "Docker - Docker Compose"
date: 2016-11-26 16:28:35 +0800
comments: true
categories: server docker
---

Docker Compose 是一個工具，用來定義與執行多個 container 組成的 Docker Applications。你可以使用 Compose 檔案來組態設定你的應用服務。然後使用單一命令，透過你的組態設定來建立與啟動你的服務。

<!-- more -->

Docker Compose 適合用來開發、測試、與建立 staging 環境，如同 CI workflows。

使用 Compose 有基本的三個處理步驟：

1. 使用 Dockerfile 定義你的 app 環境，讓它可以在任何地方都能複製(reproduced)。
2. 使用 docker-compose.yml 定義你的服務，讓他們可以在獨立環境內一起執行。
3. 最後，執行 docker-compose up，Compose 將會開始與執行你所有的 app。

```ruby
# docker-compose.yml
version: '1' # dockerfile 版本  
services:
	web:
	   container_name: exmple-web # Container 名稱
		images:"mmumshad/simple-webapp" # 使用的 Image
		ports:
			- "80:5000" # 將 container 的 port 映射到 80
	detabase:
		container_name: exmple-detabase # Container 名稱
		images:"mysql"
		volumes::
			- /opt/data:/var/lib/mysql
```

![](https://i.imgur.com/2Z22ghU.png)

```ruby
docker-compose build # 透過 docker-compose 建立好所有的 image
docker-compose up
docker-compose stop
docker-compose down
```

### Example


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

### .env file

* [Environment variables in Compose](https://docs.docker.com/compose/environment-variables/)

官方文件：

* [Get started with Docker for Mac](https://docs.docker.com/docker-for-mac/)
* [Docker Docs](https://docs.docker.com/engine/reference/builder/)
* [Docker Compose](https://docs.docker.com/compose/)
* [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/)


參考文件：

* [Docker —— 從入門到實踐](https://philipzheng.gitbooks.io/docker_practice/content/index.html)
* [淺談輕量化的虛擬技術 - Docker容器](http://www.cc.ntu.edu.tw/chinese/epaper/0036/20160321_3611.html)
* [Dockerfile簡單介紹](http://bonze.tw/%E4%BC%BA%E6%9C%8D%E5%99%A8%E7%A0%94%E7%A9%B6%E5%AE%A4/dockerfile%E7%B0%A1%E5%96%AE%E4%BB%8B%E7%B4%B9)
* [Docker Compose 初步閱讀與學習記錄](http://blog.maxkit.com.tw/2017/03/docker-compose.html)
* [Dcard 實習生活日記：小鯨魚（Docker）介紹](https://medium.com/@DcardLab/dcard-%E5%AF%A6%E7%BF%92%E7%94%9F%E6%B4%BB%E6%97%A5%E8%A8%98-%E5%B0%8F%E9%AF%A8%E9%AD%9A-docker-%E4%BB%8B%E7%B4%B9-a574b28feae4)
* [Dockerfile裡指定執行命令用ENTRYPOING和用CMD有何不同？](https://segmentfault.com/q/1010000000417103)
* [Dockerfile 的 ENTRYPOINT 與 CMD](https://beginor.github.io/2017/10/21/dockerfile-cmd-and-entripoint.html)
* [2018 年的 Rails 應用 Docker Image 包裝範例](https://5xruby.tw/posts/rails-docker-image)
* [Docker 實戰系列（一）：一步一步帶你 dockerize 你的應用](https://larrylu.blog/step-by-step-dockerize-your-app-ecd8940696f4)
* [docker-compose.yml 配置文件編寫詳解](https://blog.csdn.net/qq_36148847/article/details/79427878)
* [用30天來介紹和使用 Docker](https://ithelp.ithome.com.tw/users/20103456/ironman/1320)
* [Docker(四)：Docker 三劍客之 Docker Compose](http://www.ityouknow.com/docker/2018/03/22/docker-compose.html)

