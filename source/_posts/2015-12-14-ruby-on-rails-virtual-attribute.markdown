---
layout: post
title: "Ruby on Rails - 虛擬屬性Virtual Attribute"
date: 2015-12-14 13:13:37 +0800
comments: true
categories: rails gem
---

當要操作的屬性資料，和資料庫的欄位不相同的時候，就可以在 model 裡建立 Virtual Attribute 來取代。

###範例1 - full_name

```ruby
def full_name
    "#{self.first_name} #{self.last_name}"
end

def full_name=(value)
    self.first_name, self.last_name = value.to_s.split(" ", 2)
end
```

<!-- more -->

當資料庫欄位裡面有 `first_name` 和 `last_name` 分別去存取，但顯示的時候卻不可能只顯示一個，通常都一定是要連在一起

這時候就能夠用 Virtual Attribute 做出一個虛擬的屬性 `full_name` ， 而它的值則是 `first_name` 和 `last_name` 合在一起

這樣以後要取出名字，就只要用 `full_name` 即可，就不需要每次都在那邊相加。

###範例2 - tag_list

```ruby
def tag_list
  self.tags.map{ |t| t.name }.join(",")
end

def tag_list=(str)
  arr = str.split(",")

  self.tags = arr.map do |t|
    tag = CompanyTag.find_by_name(t)
    unless tag
      tag = CompanyTag.create!( :name => t )
    end
      tag  #沒有加這個的話，uless 最後一個回傳的會是 nil 就會爆錯
  end

end
```

取出時，先將所有 tag 透過 `map` 取出所有 name 變成一個 array，再用 `join` 轉成 string

存取時，先用 `split` 將 string 轉 array，用 name 去判斷每個元素是否已經存在資料庫，若是沒有，就 `create` 一個新的


>可以另外搭配 [select2](https://select2.github.io/) 做出比較好看的 tag !!

Rails gem
[select2-rails](https://github.com/argerim/select2-rails)

```ruby
<script>
  $("#tag_list").select2({
    tags: <%=raw Tags.all.map{|t| t.name}%>,
    tokenSeparators: [',', ' ']
  });
</script>
```


#最後來介紹一下attr_accessor

如果只要一般(沒有像上面還要取出來，單純只要讀取.存取)的虛擬屬性
（就是資料庫並不需要這個欄位，但可能要透過這個虛擬屬性，組成某個陣列裡的元素）

則可以直接用 `attr_accessor` 直接產生以下的結果

```ruby
class Girl
  def initialize(age)
    @age = age
  end
  # @age的getter方法
  def age
    @age
  end
  # @age的setter方法
  def age=(x)
    @age = x
  end
end
```

`attr_accessor` 就是可以自動產生 getter 和 setter
`attr_reader` 產生 getter
`attr_writer` 產生 setter


網路上有一篇講的很清楚，可以去看看
[Ruby 語法放大鏡之「attr_accessor 是幹嘛的?」](http://blog.eddie.com.tw/2015/03/21/attr_accessor/)

#attr_accessible 和 attr_protected

另外在網路上有發現兩個長個很相像的方法
仔細研究發現，這兩個主要好像都是為了防止 Mass assignment (大量賦值)

`attr_accessible` 被設定的 attributes `以外` 不能夠被 Mass assignment
`attr_protected ` 被設定的 attributes `自己` 不能夠被 Mass assignment

哈哈，一個是自己本身，一個是本身以外，蠻奇妙的。
原來這是在還沒有 Strong Parameter 這個功能時的防禦機制!!

詳情：

* [從 Github 被 hack，談 Rails 的安全性（ mass-assignment ）](http://blog.xdite.net/posts/2012/03/05/github-hacked-rails-security/)
* [Strong Parameter: Mass Assignment 機制的防彈衣](http://blog.xdite.net/posts/2012/08/12/strong-parameter-mass-assignment-solution)


官方文件：

* [attr_accessible](http://apidock.com/rails/ActiveRecord/Base/attr_accessible/class)
* [attr_protected](http://apidock.com/rails/ActiveRecord/Base/attr_protected/class)
* [stackoverflow](http://stackoverflow.com/questions/2652907/what-is-the-difference-between-attr-accessibleattributes-attr-protectedat)

