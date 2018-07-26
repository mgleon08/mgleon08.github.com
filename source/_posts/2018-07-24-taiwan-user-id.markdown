---
layout: post
title: "TaiwanUserID 台灣身分證字號驗證"
date: 2018-07-24 21:38:14 +0800
comments: true
categories: rails
---

你知道台灣身分證字號是怎麼算出來的嗎?

<!-- more -->

剛好公司同事有寫到，蠻有趣的來紀錄一下，要快還是要用 `while` 阿~

# 公式

```
戶籍代表的字母數字：
Ａ台北市-10 Ｂ台中市-11 Ｃ基隆市-12 Ｄ台南市-13 Ｅ高雄市-14 Ｆ台北縣-15
Ｇ宜蘭縣-16 Ｈ桃園縣-17 Ｉ嘉義市-34 Ｊ新竹縣-18 Ｋ苗栗縣-19 Ｌ台中縣-20
Ｍ南投縣-21 Ｎ彰化縣-22 Ｏ新竹市-35 Ｐ雲林縣-23 Ｑ嘉義縣-24 Ｒ台南縣-25
Ｓ高雄縣-26 Ｔ屏東縣-27 Ｕ花蓮縣-28 Ｖ台東縣-29 Ｗ金門縣-32 Ｘ澎湖縣-30
Ｙ陽明山-31 Ｚ連江縣-33

公式
A123456789 -> 10123456789

1   0   1    2    3    4    5    6    7   8   9 (拆解字母後的數字)
*   *   *    *    *    *    *    *    *   *   *
1   9   8    7    6    5    4    3    2   1   1 (固定係數)
-----------------------------------------------
1 + 0 + 8 + 14 + 18 + 20 + 20 + 18 + 14 + 8 + 9 = 130

130 % 10 == 0
```

# Code

```ruby
require 'benchmark'

LOCATION_CODE = {
  'A' => [1, 0], 'B' => [1, 1], 'C' => [1, 2], 'D' => [1, 3], 'E' => [1, 4], 'F' => [1, 5], 'G' => [1, 6], 'H' => [1, 7], 'I' => [3, 4],
  'J' => [1, 8], 'K' => [1, 9], 'L' => [2, 0], 'M' => [2, 1], 'N' => [2, 2], 'O' => [3, 5], 'P' => [2, 3], 'Q' => [2, 4], 'R' => [2, 5],
  'S' => [2, 6], 'T' => [2, 7], 'U' => [2, 8], 'V' => [2, 9], 'W' => [3, 2], 'X' => [3, 0], 'Y' => [3, 1], 'Z' => [3, 3]
}

MULTIPLIER = [1, 9, 8, 7, 6, 5, 4, 3, 2, 1, 1]

def id_card_validate(id)
  return false unless id =~ /\A[A-Z](1|2)\d{8}\z/
  chars = id.chars
  numbers = LOCATION_CODE[chars.shift] + chars.map(&:to_i)
  sum = numbers.zip(MULTIPLIER).map{ |a, b| a * b }.reduce(:+)
  # sum, i = 0, 0
  # while i <= 10
  #   sum += numbers[i] * MULTIPLIER[i]
  #   i += 1
  # end
  (sum % 10).zero?
end

n = 100000
Benchmark.bmbm do |x|
  x.report('leonji'){ n.times{ id_card_validate('A123456789') } }
end

# Rehearsal ------------------------------------------
# leonji   0.770000   0.010000   0.780000 (  0.796012)
# --------------------------------- total: 0.780000sec

#              user     system      total        real
# leonji   0.770000   0.000000   0.770000 (  0.792737)
```


參考文件

* [[轉貼] 身份證字號編碼公式說明~教你如何驗證](http://tzoyiing.pixnet.net/blog/post/29821245-%5B%E8%BD%89%E8%B2%BC%5D-%E8%BA%AB%E4%BB%BD%E8%A8%BC%E5%AD%97%E8%99%9F%E7%B7%A8%E7%A2%BC%E5%85%AC%E5%BC%8F%E8%AA%AA%E6%98%8E~%E6%95%99%E4%BD%A0%E5%A6%82%E4%BD%95%E9%A9%97)
* [台灣身份證字號驗證器](https://tonytonyjan.net/2015/04/15/national-identification-card-validator-of-taiwan/)
* [taiwanese_id_builder](https://github.com/wayne5540/taiwanese_id_builder)
