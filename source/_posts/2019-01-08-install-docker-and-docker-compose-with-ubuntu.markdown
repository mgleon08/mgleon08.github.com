---
layout: post
title: "Install Docker & Docker Compose with Ubuntu"
date: 2019-01-08 20:51:40 +0800
comments: true
categories: docker server
---

<!-- more -->

# Install Docker

### Prerequsites

Install packages to allow apt to use a repository over HTTPS:
 
 ```ruby
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
 ```
 
### Setup Docker Repository
 
 Add Docker’s official GPG key:
 
 ```ruby
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
 ```
 
After that add the Docker repository on your Ubuntu system which contains Docker packages including its dependencies
 
 ```ruby
 sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 ```
 
### Install Docker on Ubuntu
 
 Update the apt package index.
 
 ```ruby
 sudo apt-get update
 ```
 
 Install the latest version of Docker CE, or go to the next step to install a specific version:
 
 ```ruby
 sudo apt-get install -y docker-ce
 ```
 
 Verify that Docker CE is installed correctly by running the hello-world image.
 
 ```ruby
 sudo docker container run hello-world
 ```
 
# Install Docker Compose
 
Run this command to download the latest version of Docker Compose
 
 ```ruby
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
 ```
Apply executable permissions to the binary:

 ```ruby
 sudo chmod +x /usr/local/bin/docker-compose
 ```


# 免 sudo 用 docker 指令 
 
因為使用的是 `sudo` 來安裝 docker，因此在普通用戶的狀況，使用 docker 的指令會導致
 
 ```ruby
docker ps -a

# Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.39/containers/json?all=1: dial unix /var/run/docker.sock: connect: permission denied
 ```
 
可以看到最後一行 `var/run/docker.sock: connect: permission denied ` 因此來看一下這個檔案的權限狀況 
 

```ruby
ls -al /var/run/
srw-rw----  1 root docker    0 Jan 20 14:59 docker.sock
```

可以看到

* `user 權限`  rw-
* `group 權限` rw-
* `other 權限`: ---
* `擁有人`: root
* `組別`" docker

所以希望有權限，要嗎是 `root` 或是在 `docker` 這個組別裡面，先查一下目前有沒有這個組別

entire group list 

```ruby
cut -d: -f1 /etc/group | sort
```

發現沒有 `docker` 這個組別，那就自己新增一個

```ruby
sudo groupadd docker
```

將自己加入 docker group

```ruby
sudo gpasswd -a ${USER} docker
# 或是 sudo usermod -G docker -a ${USER}
```
> usemod -G 改寫用戶的組之後 如果用戶同時屬於多個組 那麼最終只會屬於一個組
> 而 gpasswd -a 則是將一個用戶加入到另一個組 同時不改變用戶原來的組

重啟 docker

```ruby
sudo service docker restart
```

切換當前 group，或重開新的 session

```ruby
newgrp - docker
```

接著就可以試著用普通用戶來執行，成功!

```ruby
docker ps -a
```

參考文件:
 
* [InstallDocker CE](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce)
* [How To Install Docker on Ubuntu 18.04 & 16.04 LTS](https://tecadmin.net/install-docker-on-ubuntu/)
* [Compose Install](https://docs.docker.com/compose/install/)
* [免sudo使用docker命令](https://www.jianshu.com/p/95e397570896)
* [Is there a command to list all Unix group names? [closed]](https://stackoverflow.com/questions/14059916/is-there-a-command-to-list-all-unix-group-names)
