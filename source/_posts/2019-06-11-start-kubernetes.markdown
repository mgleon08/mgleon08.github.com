---
layout: post
title: "Kubernetes - 認識 Kubernetes"
date: 2019-06-11 10:47:15 +0800
comments: true
categories: kubernetes
---

<!-- more -->

# 微服務架構的形成

### 傳統單體架構(Monolithic Architecture)的服務會造成巨大的不便。

1. `龐大且複雜的程式碼`: 除錯、新增功能與測試都包在一起，變得十分複雜
2. `容易以低效率的開發方式進行`: 利用新語言和新框架將更為困難，因為搬移或更動將耗費巨大成本
3. `不利於敏捷開發`: 現今的服務幾乎是每天更新，單體架構會讓這件事變得很耗時間
4. `可靠性低`: 單體架構是將所有的模組均建構在一個程序(process)內，當其中一環有bug時，容易牽一髮動全身

![monolithic-service](https://blog.gcp.expert/material/2017/02/monolithic-service.png)

### 微服務(microservices) 則將每一個具有商業邏輯的服務獨立出來。

1. 將龐大的專案拆成幾個不同面向的小專案，當程式碼夠小、容易理解、開發效率能被提高
2. 每個服務也可以依照自己的需求，選擇在不同機器上部署
3. 各團隊可以依照自己的需求使用適合自己的語言、資料庫開發
4. 各個服務之間也可獨立部署，不因一個服務癱瘓而癱瘓整個系統

![microservices](https://blog.gcp.expert/material/2017/02/microservices-768x784.png)

# Kubernetes 好處

> 協助自動化部署（automating deployment)、自動擴展（scaling)、管理容器應用程式（containerized applications)、指揮調度（Orchestration）

* 同時部署多個 containers 到一台機器上，甚至多台機器。
* 可以很容易更新容器版本並 rollback 回之前版本
* 當定義好部屬服務需求，很容易因應流量進行 Auto Scaling，可以從一台機器，延展到多台機器共同運行。
* 自動分配容器相對應的 IP 位址，透過 Service cluster 達到 load balancing 效果
* 管理各個 container 的狀態，當 Container Application 出現 crash 時，可以根據部屬定義的需求，自動偵測重啟服務

# Kubernetes 架構介紹

![](https://1.bp.blogspot.com/-VMBcuIeUCx0/W26-OBALRvI/AAAAAAAABho/ayhh3n6DgHYl_SY9CLece-B-JQs1fTq3QCLcBGAs/s1600/kubernetes%2Barchitecture%2Bexplained.jpg)

### Master Node

  1. 大總管，可做為主節點
  2. 主要負責管理叢集，協調所有在叢集的活動，像是調度應用程式、保持應用程式的狀態，擴展應用程式，以及滾動更新。
  3. 包含 Etcd, API Server, Controller Manager Server

### Node - Worker Machine

  1. 主要工作的節點，上面運行了許多容器
  2. 一個 Node 是一個 VM 或一個實體的主機  
  3. 一台虛擬機，K8S可操控高達1,000個nodes以上
  4. 包含 Kubelet, Proxy, Pod, Container
  5. 一個 Node 可能有一個或者是多個 Pod

  7. 每個 Node 都會執行一個 Container 的 Runtime，主要是拉取 Image 並執行 Container


> masters和nodes組成叢集(Clusters)

### iptables

1. 每個 Node 都有屬於它自己的 iptables，iptables 是 Linux 上的防火牆(firewall)，不只限制哪些連線可以連進來，也會管理網路連線，決定收到的 request 要交給哪個 Pod

### Kubelet

1. Node 上都會有 kubelet，用於管理該 Node 上的所有 pods 以及與 master node 即時溝通

### Kube-proxy

1. kube-proxy 會將目前該 Node 上所有 Pods 的資訊傳給 iptables，讓 iptables 即時獲得在該 Node 上所有 Pod 的最新狀態
2. 舉例當一個 Pod 物件被建立時，kube-proxy 會通知 iptables，以確保該 Pod 可以被 Kubernetes Cluster 中的其他物件存取。

![](https://github.com/zxcvbnius/k8s-30-day-sharing/blob/master/Day06/kubernetes-orverview.png?raw=true)

### Pod

> 容器是位於 pod 內部，一個 pod 包覆著一個以上的 Container，這造成 K8S 與一般容器不同的操作概念。在Docker裡，Docker container 是最小單位。

1. Pod 是 Kubernetes 裡面最小的單位
2. 在 Pod 裡面可以有一個或多個 container，並共享相同資源，Volume、Storage、Network 資源以及配置設定
3. Pod 採取shared IP，內部所有的 container 皆使用同一個Pod IP，因此可以互相溝通，這也與Docker不同
4. Pod 內的 container 就像是生命共同體，一旦 Pod 被調度，所有 Pod 內的 container 也會一起被調度
5. 一個給定的 Pod（UID 被定義）不會被調度到新的 Node 上，而是由一個全新相同而且帶著不同的 UID Pod 取代
6. Pod 在建立後，會擁有一個 Unique ID
7. Pod 擁有不確定的 [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)，這意味著您不曉得任一pod是否會永久保留

### Service

> Service 元件作為一個橋樑，讓 Cluster 以外的服務也可以與 Pod 做互動。

1. 每個Service包含著一個以上的pod
2. 每個Service有個獨立且固定的IP地址 – Cluster IP，不會因為 Pod 關閉重啟而喪失原來的 IP 位置。
3. 客戶端訪問Service時，會經由上述提過的proxy來達到負載平衡、與各pod連結的結果
4. 利用標籤選擇器(Label Selector)，聰明地選擇那些已貼上標籤的pod

### Deployments (舊版為 Replication Controller)

> Deployments顧名思義掌控了部署Kubernetes服務的一切。它主要掌管了Replica Set的個數，而Replica Set的組成就是一個以上的Pod。

1. Deployments 的設定檔(底下以YAML格式為例)，可以指定replica，並保證在該replica的數量運作
2. Deployments 會檢查pod的狀態
3. Deployments 下可執行滾動更新或者回滾
4. Deployment 為管理 Pod 的 Controller，可視一組 Deployment 為一組應用服務

```yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

# Reference

* [Kubernetes 初體驗 By minikube](https://neighborhood999.github.io/2018/08/27/learing-k8s-by-minikube/)
* [Kubernetes 30天學習筆記](https://ithelp.ithome.com.tw/users/20103753/ironman/1590)
* [Kubernetes 與 minikube 入門教學](https://blog.techbridge.cc/2018/12/01/kubernetes101-introduction-tutorial/)
* [GKE 系列文章(一) – 為什麼使用 Kubernetes](https://blog.gcp.expert/kubernetes-gke-introduction/)
