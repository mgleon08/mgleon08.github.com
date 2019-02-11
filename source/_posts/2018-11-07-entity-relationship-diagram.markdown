---
layout: post
title: "Entity Relationship Diagram (ERD)"
date: 2018-11-07 15:14:59 +0800
comments: true
categories: diagram other
---

在工作上時常用到 Entity Relationship Diagram (ERD) 來規劃 DB schema 的關聯，透過這個可以在規劃專案時很清楚知道整個架構，並且讓執行的人也可以很清楚要怎麼實作。

<!-- more -->

畫出來的圖大概會長這樣

![](https://i.imgur.com/aGA5DHb.png)

這邊的重點就是要瞭解中間關聯的線的左右邊代表了什麼意思 (如以下圖)

![ERPD Cardinality](https://d2slcw3kip6qmk.cloudfront.net/marketing/pages/chart/erd-symbols/ERD-Notation.PNG)

以第一張圖舉例來說就代表

* 1 個 customer 可以有 0 個或多個 order
* 1 個 order 至少有 1 個或多個 product
* 1 個 product 可以有 0 個或多個 order
* 1 個 order 至少且只能有一個 customer

更進階清楚的話，可以連所有 column 的 type 都寫進去

![](https://i.imgur.com/KBUmyAK.png)

* [Entity Relationship Diagram (ERD) Tutorial - Part 1](https://www.youtube.com/watch?v=QpdhBUYk7Kk)
* [Entity Relationship Diagram (ERD) Tutorial - Part 2](https://www.youtube.com/watch?v=-CuY5ADwn24)
* [Entity Relationship Diagram](https://www.smartdraw.com/entity-relationship-diagram/)
