---
layout: post
title: "UML Class Diagram"
date: 2018-11-08 15:15:11 +0800
comments: true
categories: diagram
---

類別圖，跟 ERD 有些類似，可能包含了 ERD 所描述的內容，但不一定可對應

<!-- more -->

Unified Modeling Language (UML)，透過此圖可以很清楚的畫出，每個 model 裡面有的 `attributes` & `methods`，還有跟別的 model 的相互關係

### Symbol

每個 `attributes` & `methods` 前面可以標示符號，代表這個目前是哪一種狀態

* private : -
* public : +
* protected : #
* package / default : ~

![](https://i.imgur.com/gkyiOkn.png)

上圖算是比較簡易的版本，有些在 method 後面還會加入傳入的參數，跟回傳值

```ruby
- setName(string): string
- eat(string): string
```

### Relationship

透過 model 跟 model 間線的關聯，可以知道是繼承還是組合等等

* Inheritance 繼承 –––––▷
* Association 關聯 ––––––
* Aggregation 聚合 –––––◆
* Composition 組合 –––––◇
* Dependency 依賴  ----->

### Multiplicity

線上面的數字則可以表示，model 與 model 的對應關係

* `0..1` zero to one (optional)
* `n` specific number
* `0..*` zero to many
* `1..*` one to many
* `m..n` specific number range

### Example

實際上大概的圖示

![](https://i.imgur.com/B6mapCR.png)

* [UML Class Diagram Tutorial](https://www.youtube.com/watch?v=UI6lqHOVHic)
* [[軟工] 類別圖「關聯」、「聚合」及「組合」比較](https://dotblogs.com.tw/lazycodestyle/2016/06/01/233545)
* [UML超新手入門](https://github.com/macdidi5/UMLTutorial)
