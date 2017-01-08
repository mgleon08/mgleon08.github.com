---
layout: post
title: "Coding Principle 程式設計原則 SOLID"
date: 2016-05-29 20:27:06 +0800
comments: true
categories: clean_code
---

在 Coding 的世界中，有許多重要的 Principle 值得我們去遵循

<!-- more -->
#SOLID

#Single Responsibility Principle(單一責任原則SRP)
###定義：
* 對一個類別而言，應該僅有一個引起它變化的原因（職責）
* 降低單一類別被「改變」所影響的機會
* 只有一個理由需要更改這個class，如果有一個以上的理由就表示：這個class負責超過一個以上的責任

###說明：
* 若一個類別有多重職責，職責之間會互相耦合，一個職責的變化可能會影響該類別完成其他職責的能力。


[軟體路上不孤單Day10-物件導向原則介紹3[SRP]](http://ithelp.ithome.com.tw/articles/10100557)

#Open/Close Principle (開放關閉原則OCP)
###定義：
* 軟體模組（class, method, module）應該對擴展開放，對修改關閉
* 讓模組容易增加(擴展)功能，而不必去修改原有程式碼
* 讓主要類別不會因為新增需求而改變

###說明：
* 對有相似行為類別的建立抽象層，如 abstract class, 或是 interface。
* 將公共屬性或方法提取到抽象層中，當需要擴展行為（新增）時只需要建立新的子類別並繼承抽象層，不必修改原有的行為。

>注意：實作 OCP 抽象層需要花費時間和精力，也可能會造成複雜度的上升，OCP 應該只運用在程序中頻繁發生的變化上。

[軟體路上不孤單Day08-物件導向原則介紹1[OCP]](http://ithelp.ithome.com.tw/articles/10100008)

#Liskov Substitution Principle (Liskov替換原則LSP)
###定義：
* 子類別(Sub Type)必須能夠替換成他們的基本類別(Base Type)
* 子類別應該可以替換任何基本類別出現的位置，且程序還能正常工作
* 避免繼承時子類別所造成的「行為改變」

###說明：
1. 不能僅用 is-a 的關係就建立繼承，必須考慮是否在基本類別中有些方法對子類別而言是不需要或是無意義的。

這些沒有意義的方法會造成不可預期的結果。

2. 建立一個抽象層並提取公共方法，並讓子類別派生抽象層。

>注意：必須要有繼承關係才需要考慮 LSP ，而 LSP 是讓設計達到 OCP 的規則之一 。

[軟體路上不孤單Day11-物件導向原則介紹4[LSP]](http://ithelp.ithome.com.tw/articles/10100827)

#Law of Demeter(迪米特原則LOD, LKD)

###定義：
* 也稱最少知識原則(Principle of Least Knowledge)
* 避免曝露過多資訊造成用戶端因流程調整而改變
* 模組應該儘可能的減少其他模組交互，目的在於降低彼此之間的依賴。

###說明：
以下為必須遵循 LOD 的條件

類別 O 的任何方法 m 只能呼叫屬於以下情況的方法

1. 類別 O 本身的方法
2. 傳入 m 的參數的方法
3. 在 m 中建立對象的方法
4. 任何直接持有的對象方法

[軟體路上不孤單Day13-物件導向原則介紹6[LoD]](http://ithelp.ithome.com.tw/articles/10101265)

#Interface Segregation Principle(介面隔離ISP)
###定義：
* 使用單純簡單的 interface , 比使用一個複雜過大的 interface 來的好。   
* 用戶不應該被迫相依於他們用不到的函示
* 降低用戶端因為不相關介面而被改變  

###說明：
一個過大的 interface ，通常代表其中有某些功能是客戶端不需要的，如果客戶端實作了不需要的功能 ，這些功能會造成不必要的耦合。

我們可以把過大的 interface 分離，將其中某些功能拆離到另一個 interface 中。

[軟體路上不孤單Day12-物件導向原則介紹5[ISP]](http://ithelp.ithome.com.tw/articles/10101106)

#Dependency Inversion Principle (相依性反轉DIP)

###定義：
* 高層模組不應該相依於低層模組，兩者都應該相依於抽象
* 避免高階程式因為低階程式改變而被迫改變
* 抽象不應該相依於具體，具體應該相依於抽象。

###說明：
對象的引用盡量是抽象型態而不是具體型態。

>注意：若是具體型態已經相當穩定，不太會變化，依賴於該具體類別也是無妨。

[軟體路上不孤單Day14-物件導向原則介紹7[DIP]](http://ithelp.ithome.com.tw/articles/10101486)

---

#Do not repeat yourself(DRY)
###定義：
* 當有相同的 code 時，應該整合成一個，不該重複出現

[軟體路上不孤單Day09-物件導向原則介紹2[DRY]](http://ithelp.ithome.com.tw/articles/10100309)

#duck type(鴨子類型)
###定義：
* 當我看到一隻鳥，它走路像鴨子，游泳像鴨子，叫聲像鴨子，我就稱其為鴨子

###說明：
```ruby
def hello(x)
    x.hi
end
```

因此只要 x 將帶有 `hi` 的方法都可以使用，不管是人或是動物

[進一步思考Duck typing](http://www.ithome.com.tw/voice/88063)

參考文件： 
 
* [白話- OO設計原則 (SOLID原則) - 附生活實例](http://rockssdlog.blogspot.tw/2012/03/oo-solid.html)   
* [軟體路上不孤單](http://ithelp.ithome.com.tw/search?tab=article&search=%E8%BB%9F%E9%AB%94%E8%B7%AF%E4%B8%8A%E4%B8%8D%E5%AD%A4%E5%96%AE&page=1)  
* [Design Principle](http://122.146.238.121/wordpress/?cat=95)  
* [從實例學設計模式](http://slides.com/jaceju/design-patterns-by-examples/#/)
* [Bassist 中勝](https://blog.jason.party/)