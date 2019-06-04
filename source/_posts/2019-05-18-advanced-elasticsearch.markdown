---
layout: post
title: "Advance ElasticSearch"
date: 2019-05-18 08:55:02 +0800
comments: true
categories: elasticsearch
---

<!-- more -->

# Bool Query 

* `must` - 查詢必須匹配的字，並計算 `_score` (與 AND 等價)
* `filter` - 查詢必須匹配的字，不計算 `_score` (代表對評分沒有任何貢獻，只是用來過濾)
* `should` - 滿足任一匹配的字，將增加 `_score` ，否則，無任何影響，如果一個 query 中沒有 `must` 和 `filter` 則必須匹配一個或以上的 `should` (與 OR 等價)
* `must_not` - 查詢排除的字 (與 NOT 等價)
* `boost` - 權重
* `minimum_should_match` - 設定 `should` 至少要匹配幾個句子


## Example 1:

user 必須是 kimchy，並且過濾出 tag 是 "tech" (匹配多寡並不影響 _score)，age 範圍排除 10 ~ 20，如果 tag 有 `wow` 或是 `elasticsearch` 則 _score 比較高，兩個都有則更高

```go
{
    "bool" : {
        "must" : {
            "term" : { "user" : "kimchy" }
        },
        "filter": {
            "term" : { "tag" : "tech" }
        },
        "must_not" : {
            "range" : {
                "age" : { "from" : 10, "to" : 20 }
            }
        },
        "should" : [
            {
                "term" : { "tag" : "wow" }
            },
            {
                "term" : { "tag" : "elasticsearch" }
            }
        ],
        "minimum_should_match" : 1,
        "boost" : 1.0
    }
}
```


## Example 2:

> 將 bool 帶入 filter 一樣可以不計算分數

查找 `title` 字段匹配 `how to make millions` 並且不被 `tag` 為 `spam` 的文件。那些被 `tag` 為 `starred` 或在2014之後的文件，將比另外那些文件擁有更高的排名。如果 _兩者_ 都滿足，那麼它排名將更高，並過濾出 `price` 必須小於等於 29.99，且 `category` 不能是 `ebooks` 這兩個條件則不影響排名

```go
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ],
        "filter": {
          "bool": { 
              "must": [
                  { "range": { "price": { "lte": 29.99 }}}
              ],
              "must_not": [
                  { "term": { "category": "ebooks" }}
              ]
          }
        }
    }
}
```

## Example3: constant_score

它將一個不變的常量評分應用於所有匹配的文件，比較簡潔用來取代只有一個 filter 的 bool

```go
{
    "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" } 
        }
    }
}
```

## Example4: boost 權重

預設為 1

```go
{
    "query": {
        "bool": {
            "must": {
                "match": {  
                    "content": {
                        "query":    "full text search",
                        "operator": "and"
                    }
                }
            },
            "should": [
                { "match": {
                    "content": {
                        "query": "Elasticsearch",
                        "boost": 3 
                    }
                }},
                { "match": {
                    "content": {
                        "query": "Lucene",
                        "boost": 2 
                    }
                }}
            ]
        }
    }
}
```

## Example5: equle to match

### OR

下面兩個相等

```go
{
    "match": { "title": "brown fox"}
}
```

```go
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
```

### AND

下面兩個相等

```go
{
    "match": {
        "title": {
            "query":    "brown fox",
            "operator": "and"
        }
    }
}
```

```go
{
  "bool": {
    "must": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }}
    ]
  }
}
```

### minimum_should_match

下面兩個相等

```go
{
    "match": {
        "title": {
            "query":                "quick brown fox",
            "minimum_should_match": "75%"
        }
    }
}
```

```go
{
  "bool": {
    "should": [
      { "term": { "title": "brown" }},
      { "term": { "title": "fox"   }},
      { "term": { "title": "quick" }}
    ],
    "minimum_should_match": 2 
  }
}
```


# Exact Values Search

### 數字查詢

```sql
SELECT document
FROM   products
WHERE  price = 20
```

通常精準的字查詢，就不需要計算分數，因此加上 `constant_score`

```go
{
    "query" : {
        "constant_score" : { 
            "filter" : {
                "term" : { 
                    "price" : 20
                }
            }
        }
    }
}
```

### text 查詢

```sql
SELECT product
FROM   products
WHERE  productID = "XHDK-A-1293-#fJ3"
```

這裡會有個問題，分析器會解析 `XHDK-A-1293-#fJ3` -> `XHDK` `A` `1293` `#fJ3`，因此查詢時會有問題

```go
GET /my_store/products/_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "productID" : "XHDK-A-1293-#fJ3"
                }
            }
        }
    }
}
```

必須重新針對 productID 設定不要分析，重新設定前記得先刪除原本的 index

```go
{
    "mappings" : {
        "products" : {
            "properties" : {
                "productID" : {
                    "type" : "string",
                    "index" : "not_analyzed" 
                }
            }
        }
    }

}
```

# Combining Filters

Example1

```sql
SELECT product
FROM   products
WHERE  (price = 20 OR productID = "XHDK-A-1293-#fJ3")
  AND  (price != 30)
```

```go
{
   "query" : {
      "bool" : { 
         "filter" : {
            "bool" : {
              "should" : [
                 { "term" : {"price" : 20}}, 
                 { "term" : {"productID" : "XHDK-A-1293-#fJ3"}} 
              ],
              "must_not" : {
                 "term" : {"price" : 30} 
              }
           }
         }
      }
   }
}
```

Example2

```sql
SELECT document
FROM   products
WHERE  productID      = "KDKE-B-9947-#kL5"
  OR (     productID = "JODL-X-1937-#pV7"
       AND price     = 30 )
```

```go
{
   "query" : {
      "bool" : {
         "filter" : {
            "bool" : {
              "should" : [
                { "term" : {"productID" : "KDKE-B-9947-#kL5"}}, 
                { "bool" : { 
                  "must" : [
                    { "term" : {"productID" : "JODL-X-1937-#pV7"}}, 
                    { "term" : {"price" : 30}} 
                  ]
                }}
              ]
           }
         }
      }
   }
}
```

Example3

* 在收件箱中，且沒有被讀過的
* 不在 收件箱中，但被標註重要的

```go
{
  "query": {
      "constant_score": {
          "filter": {
              "bool": {
                 "should": [
                    { "bool": {
                          "must": [
                             { "term": { "folder": "inbox" }}, 
                             { "term": { "read": false }}
                          ]
                    }},
                    { "bool": {
                          "must_not": {
                             "term": { "folder": "inbox" } 
                          },
                          "must": {
                             "term": { "important": true }
                          }
                    }}
                 ]
              }
            }
        }
    }
}
```

# Disjunction Max Query 最佳字段

給予兩個字段

```go
PUT /my_index/my_type/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}

PUT /my_index/my_type/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}
```

### bool

使用一般的 bool，會發現 1 的分數會比較高，主要在於 1 的兩個句字都有包含到 `Brown`，但我們希望的是比較準確的 2，因為 body 就包含了 `Brown fox`

```go
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```


### dis_max

將任何與任一查詢匹配的文件作為結果返回，但只將最佳匹配的評分作為查詢的評分結果返回

```go
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

如果用 dis_max 查出的兩個最佳匹配分數一樣，可以加上 tie_breaker 調優，將其他匹配的語句一起做計算並乘個比例，範圍在 0~1

> tie_breaker 可以是 0 到 1 之間的浮點數，其中 0 代表使用 dis_max 最佳匹配語句的普通邏輯， 1 表示所有匹配語句同等重要。最佳的精確值需要根據數據與查詢調試得出，但是合理值應該與零接近（處於 0.1 - 0.4 之間），這樣就不會顛覆 dis_max 最佳匹配性質的根本。

```go
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            "tie_breaker": 0.3
        }
    }
}
```

# Function Score Query

* [透過Function Score Query優化Elasticsearch搜索結果](https://www.scienjus.com/elasticsearch-function-score-query/)
* [按受歡迎度提升權重](https://www.elastic.co/guide/cn/elasticsearch/guide/cn/boosting-by-popularity.html)
* [過濾集提升權重](https://www.elastic.co/guide/cn/elasticsearch/guide/cn/function-score-filters.html)

# Reference

* [Elasticsearch: 權威指南 \| Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)
* [[Elasticsearch] 全文搜索 (三) - match查询和bool查询的关系，提升查询子句 - dm_vincent的专栏 - CSDN博客](https://blog.csdn.net/dm_vincent/article/details/41743955)
* [Elasticsearch DSL 常用語法介紹](https://cloud.tencent.com/developer/article/1113818)
