---
layout: post
title: "Docker - Docker Machine, Docker Swarm"
date: 2018-12-28 23:14:00 +0800
comments: true
categories: server docker
---
 
透過 docker machine 在 local 測試 docker swarm!
雖然目前是 k8s 比較紅，但還是先來玩玩看 swarm，之後再來研究 k8s (Kubernetes)

<!-- more -->

## Docker Machine

Docker Machine 最主要的目的就是可以快速的把虛擬機和 Docker 環境架設起來。

![](https://docs.docker.com/machine/img/machine.png)

```ruby
# 建立, --driver=virtualbox
docker-machine create -d virtualbox machine1

#啟動
docker-machine start machine1

# 停止
docker-machine stop machine1

# 重啟
docker-machine restart machine1

# 刪除
docker-machine rm machine1

# list
docker-machine ls

# ssh
docker-machine ssh machine1

#檢查訊息
docker-machine inspect

#查看狀態
docker-machine status machine1
```

## Docker Swarm

![](https://docs.docker.com/engine/swarm/images/ingress-routing-mesh.png)

* 運行 docker 的主機可初始化或加入 Swarm 集群，這樣主機就成為 Swarm 集群的節點 (node)。
* 節點分為管理 (manager 管理成員以及指派任務) 節點和工作 (worker 執行 swarm service) 節點。
* 可以有多個 manager 但只能有一個 leader
* 管理節點用於 Swarm 集群的管理，docker swarm 命令基本只能在管理節點執行（節點退出集群命令 docker swarm leave 可以在工作節點執行）。
* 一個 Swarm 集群可以有多個管理節點，但只有一個管理節點可以成為 leader。


> 由於 docker swarm 最主要是多台機器可以在同一個群集，但在 local 並沒有那個多台機器，因此可以透過 docker machine 的方式，建立多台 vm，來模擬多台機器的狀況

一開始記得先安裝 [virtualbox](https://www.virtualbox.org)

執行 

```ruby
brew cask install virtualbox
```

建立用來當 manager 的機器，如果沒有 Boot2Docker 就會自動去安裝

```ruby
# docker-machine create --driver virtualbox manager1
# 由於新版會有問題，所以改指定版本
docker-machine create --virtualbox-boot2docker-url https://github.com/boot2docker/boot2docker/releases/download/v18.06.1-ce/boot2docker.iso  --driver virtualbox manager1
```
* [18.09.0 iso breaks swarm ingress](https://github.com/docker/machine/issues/4608)

```ruby
Running pre-create checks...
(manager1) Image cache directory does not exist, creating it at /Users/leon/.docker/machine/cache...
(manager1) No default Boot2Docker ISO found locally, downloading the latest release...
(manager1) Latest release for github.com/boot2docker/boot2docker is v18.09.0
(manager1) Downloading /Users/leon/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.09.0/boot2docker.iso...
(manager1) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(manager1) Unable to get the local Boot2Docker ISO version:  Did not find prefix "-v" in version string
(manager1) Default Boot2Docker ISO is out-of-date, downloading the latest release...
(manager1) Latest release for github.com/boot2docker/boot2docker is v18.09.0
(manager1) Downloading /Users/leon/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.09.0/boot2docker.iso...
(manager1) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
(manager1) Copying /Users/leon/.docker/machine/cache/boot2docker.iso to /Users/leon/.docker/machine/machines/manager1/boot2docker.iso...
(manager1) Creating VirtualBox VM...
(manager1) Creating SSH key...
(manager1) Starting the VM...
(manager1) Check network to re-create if needed...
(manager1) Found a new host-only adapter: "vboxnet0"
(manager1) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env manager1
```

接著查看 ip

```ruby
docker-machine env manager1
```
```ruby
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/leon/.docker/machine/machines/manager1"
export DOCKER_MACHINE_NAME="manager1"
# Run this command to configure your shell:
# eval $(docker-machine env manager1)
```

再新增兩台機器中當作 worker

```ruby
docker-machine create --virtualbox-boot2docker-url https://github.com/boot2docker/boot2docker/releases/download/v18.06.1-ce/boot2docker.iso  --driver virtualbox worker1
docker-machine create --virtualbox-boot2docker-url https://github.com/boot2docker/boot2docker/releases/download/v18.06.1-ce/boot2docker.iso  --driver virtualbox worker2
```

### 接著建立 docker swarm 集群

![](https://docs.docker.com/engine/swarm/images/swarm-diagram.png)

進入 manager

```ruby
docker-machine ssh manager1
```

將 manager1 指定為 manager

```ruby
docker swarm init --advertise-addr 192.168.99.100
```
```ruby
Swarm initialized: current node (2vo249yn5wa5inamg7r20hu1q) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4vl3brd3z91s6kqsqjhz4p9dqnesanq30bu2kgty1krin8e5wp-0yd8lyh4g1l30njzo2yxflcjo 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

進入 worker1 & worker2

```ruby
docker-machine ssh worker1
docker-machine ssh worker2
```

加入 swarm 

```ruby
 docker swarm join --token SWMTKN-1-4vl3brd3z91s6kqsqjhz4p9dqnesanq30bu2kgty1krin8e5wp-0yd8lyh4g1l30njzo2yxflcjo 192.168.99.100:2377
```

接著到 manager 看目前 node 狀況，只有 manager1 是 leader

```ruby
docker node ls
```
```ruby
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
2vo249yn5wa5inamg7r20hu1q *   manager1            Ready               Active              Leader              18.09.0
jhv6fv23627gpja6nhu06bx1a     worker1             Ready               Active                                  18.09.0
80x4t8oyhh7orpck7uksuv24c     worker2             Ready               Active                                  18.09.0
```

網路狀況，可看到 swarm 是 overlay

```ruby
docker network ls
```
```ruby
NETWORK ID          NAME                DRIVER              SCOPE
f01fa41ff7a6        bridge              bridge              local
1979e800b7d0        docker_gwbridge     bridge              local
85fd158c6bb0        host                host                local
jpgat6x8ukxl        ingress             overlay             swarm
4e106b8319a7        none                null                local
```

進到 manager1 啟動一個 service

```ruby
docker service create --replicas 1 -d -p 8080:80 --name my_nginx nginx
```

查看跑在哪個 worker

```ruby
docker service ps my_nginx
```
```ruby
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
lu4in7orcr7t        my_nginx.1          nginx:latest        worker2             Running             Running 9 minutes ago
```

查看詳細資料

```ruby
docker service inspect --pretty my_nginx
```

接著就可以在

```ruby
# 看到所有 nginx
192.168.99.100:8080
192.168.99.101:8080
192.168.99.102:8080

# 可以試試看更改 /usr/share/nginx/html/index.html
```

調整 replicas

```ruby
docker service update --replicas 2 my_nginx
```


### 相關指令

```ruby
# 查看 ip 等相關訊息
docker-machine env manager1

# 初始化一個 Swarm 集群(manager, leader)
docker swarm init --advertise-addr 192.168.99.100

# 其他 worker join 到同一個集群
docker swarm join --token [token] [master的IP]:[master的port]

# 獲取加入集群的token
docker swarm join-token worker

# 取得 manager toke，可以用另外一台當 manager 的職代，可以有多個 manager 但只能有一個 leader
docker swarm join-token manager

# 離開 swarm
docker swarm leave manager

# 查看 node 資料
docker node inspect

# 移除 node
docker node rm worker1

# 讓其中2份nginx容器副本，分配到集群中
docker service create --replicas 2 -d -p 8080:80 --name my_nginx nginx

# 查看
docker service ls

# 查看目前跑在哪個 service
docker service ps my_nginx
docker service inspect --pretty my_nginx

# 擴容service中的任務，到第三台
docker service scale my_nginx=3

# remove service
docker service rm my_nginx
```

參考文件: 

* [Docker Machine Overview](https://docs.docker.com/machine/overview/)
* [Swarm mode overview](https://docs.docker.com/engine/swarm/)
* [docker service](https://docs.docker.com/engine/reference/commandline/service/)
* [Use swarm mode routing mesh](https://docs.docker.com/engine/swarm/ingress/)
* [docker-swarm-tutorial](https://github.com/twtrubiks/docker-swarm-tutorial)
* [Docker(五)：Docker 三剑客之 Docker Machine](http://www.ityouknow.com/docker/2018/03/30/docker-machine.html)
* [Docker(六)：Docker 三劍客之 Docker Swarm](http://www.ityouknow.com/docker/2018/04/19/docker-swarm.html)
* [Docker Swarm集群初探](https://www.jianshu.com/p/3f3c9e0e3db5)
* [Docker Swarm 入門一篇文章就夠了](https://www.jianshu.com/p/9eb9995884a5)
