---
layout: post
title: "Docker - clean"
date: 2020-03-12 22:37:49 +0800
comments: true
categories: docker
---

移除一些未使用佔空間的檔案

<!-- more -->

```
docker image prune
docker volume prune
docker system prune
```

快速刪除

```
# container
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)

# image
docker rmi $(docker image list -q)

# volume
docker volume rm $(docker volume ls -qf dangling=true)
```


Reference:

* [docker cleanup guide: containers, images, volumes, networks · GitHub](https://gist.github.com/bastman/5b57ddb3c11942094f8d0a97d461b430)
