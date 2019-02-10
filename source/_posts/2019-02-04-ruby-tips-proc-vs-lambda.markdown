---
layout: post
title: "Ruby Tips - proc vs lambda"
date: 2019-02-04 17:17:43 +0800
comments: true
categories: ruby ruby_tips
---

<!-- more -->

* lambda 會檢查參數的個數

```ruby
def args(code)
  code.call(1, 2, 3, 4)
end

# 也可以寫 proc { |x, y, z| x + y + z }
proc = Proc.new { |x, y, z| x + y + z }
lambda = lambda { |x, y, z| x + y + z }

args(proc)
# => 6
args(lambda)
#ArgumentError: wrong number of arguments (4 for 3)
```
* lambda 的 return 會繼續執行，proc 則會直接終止

```ruby
def proc_return
  Proc.new { return "Proc.new" }.call
  return "proc_return method finished"
end

def lambda_return
  # 也可以寫成 lambda { return "lambda" }
  -> { return "lambda" }.call
  return "lambda_return method finished"
end

proc_return
# => "Proc.new"
lambda_return
# => "lambda_return method finished"
```

```ruby
def generic_return(code)
  one, two    = 1, 2
  three, four = code.call(one, two)
  "#{three} and #{four}"
end

generic_return(-> (x, y){ return x + 1, y + 2 })
# 也可以寫成 lambda { |x, y| return x + 1, y + 2 }
#  => "2 and 4"
generic_return(Proc.new { |x, y| return x + 1, y + 2 })
# LocalJumpError: unexpected return
generic_return(Proc.new { |x, y| x + 1; y + 2 })
#  => "4 and "
generic_return(Proc.new { |x, y| [x + 1, y + 2] })
#  => "2 and 4"
```

參考文件

* [Ruby 中的 Block & Yield & Proc & Lambda & Method]({{ root_url }}/blog/2016/02/06/block-yield-proc-lambda/)
