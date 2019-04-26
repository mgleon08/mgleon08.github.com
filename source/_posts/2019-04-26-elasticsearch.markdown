---
layout: post
title: "Getting started with the ElasticSearch"
date: 2019-04-26 23:36:58 +0800
comments: true
categories: elasticsearch
---

<!-- more -->

Elasticsearch is a distributed, RESTful search and analytics engine capable of solving a growing number of use cases. As the heart of the Elastic Stack, it centrally stores your data so you can discover the expected and uncover the unexpected.

## Elasticsearch vs RDBMS

* Node - Server
* Indices - Databases
* Types - Tables
* Documents - Rows
* Fields - Columns

> After 6.0 version need add `-H 'Content-Type:application/json'`

## Install

java

```go
brew cask install java
brew cask install homebrew/cask-versions/java8
```

elasticsearch

```go
brew install elasticsearch24
```

## Check

```go
curl localhost:9200
```

```go
{
  "name" : "Crimson Daffodil",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "_1W7Qb8WSByw_AoBtx3V9g",
  "version" : {
    "number" : "2.4.6",
    "build_hash" : "5376dca9f70f3abef96a77f4bb22720ace8240fd",
    "build_timestamp" : "2017-07-18T12:17:44Z",
    "build_snapshot" : false,
    "lucene_version" : "5.5.4"
  },
  "tagline" : "You Know, for Search"
}
```

### health

```go
curl localhost:9200/_cluster/health?pretty=true
```

```go
{
  "cluster_name" : "elasticsearch_leon",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 10,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}
```

## Count

```go
curl -X GET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'
```

```go
{
  "count" : 0,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  }
}
```

## Show all Index

```go
curl -X GET 'http://localhost:9200/_cat/indices?v'
```

```go
health status index    pri rep docs.count docs.deleted store.size pri.store.size 
yellow open   accounts   5   1          1            0      4.1kb          4.1kb 
```

## Show all Index mapping type

```go
curl 'localhost:9200/_mapping?pretty=true'
```

```go
{
  "accounts" : {
    "mappings" : {
      "person" : {
        "properties" : {
          "desc" : {
            "type" : "string"
          },
          "title" : {
            "type" : "string"
          },
          "user" : {
            "type" : "string"
          }
        }
      }
    }
  }
}
```

## Create Index

```go
curl -X PUT 'localhost:9200/weather'
```

```go
{
  "acknowledged":true
}
```

## Delete Index

```go
curl -X DELETE 'localhost:9200/weather'
```

```go
{
  "acknowledged":true
}
```

## Setting Analyzer

[elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik/)

```go
$ curl -X PUT 'localhost:9200/accounts' -d '
{
  "mappings": {
    "person": {
      "properties": {
        "user": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "categories": {
           "type": "nested",
           "properties": {
              "name": {
                "type": "string"
              }
           }
        }
      }
    }
  }
}'
```

## Create Document

with id, also can string

```go
// can replace 1 to first
curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
  "user": "leon",
  "title": "工程師",
  "desc": "數據庫管理",
  "age": 18
}' 
```

```go
{
  "_index": "accounts",
  "_type": "person",
  "_id": "1",
  "_version": 1,
  "_score": 1,
  "_source": {
    "user": "leon",
    "title": "工程師",
    "desc": "數據庫管理",
    "age": 18
  }
}
```

without id will auto create random uuid

```go
curl -X POST 'localhost:9200/accounts/person' -d '
{
  "user": "mark",
  "title": "工程師",
  "desc": "系統管理",
  "age": 28
}'
```

## Show

```go
curl 'localhost:9200/accounts/person/1?pretty=true'

// _source without other data
curl 'localhost:9200/accounts/person/1/_source?pretty=true'
```

```go
{
  "_index": "accounts",
  "_type": "person",
  "_id": "AWpP4wxUKXQ2yoYJjuiA",
  "_version": 1,
  "found": true,
    "_source": {
      "user": "leon",
      "title": "工程師",
      "desc": "數據庫管理",
      "age": 20
    }
}
```

not found

```go
curl 'localhost:9200/weather/beijing/abc?pretty=true'
```

```go
{
  "_index" : "weather",
  "_type" : "beijing",
  "_id" : "abc",
  "found" : false
}
```


## Delete

```go
curl -X DELETE 'localhost:9200/accounts/person/1'
```

## Reindex (not Update)

elastic can't update, only create new, and increase the version number

```go
curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
    "user" : "leon",
    "title" : "工程師",
    "desc" : "數據庫管理，軟件開發",
    "age": 20
}' 
```

version was change to 2

```go
{
  "_index": "accounts",
  "_type": "person",
  "_id": "1",
  "_version": 2,
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "created": false
}
```

## Show all documents (empty search)

```go
curl 'localhost:9200/accounts/person/_search?pretty=true'
```

* took - search time
* hits - hit how many record
* _score - match score

```go
{
  "took" : 50,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "accounts",
      "_type" : "person",
      "_id" : "2",
      "_score" : 1.0,
      "_source" : {
        "user" : "leon",
        "title" : "工程師",
        "desc" : "數據庫管理",
        "age": 20
      }
    }, {
      "_index" : "accounts",
      "_type" : "person",
      "_id" : "1",
      "_score" : 1.0,
      "_source" : {
        "user" : "mark",
        "title" : "工程師",
        "desc" : "數據庫管理，軟件開發",
        "age": 28
      }
    } ]
  }
}
```

## `filtered` Search

```go
curl 'localhost:9200/accounts/person/_search?pretty=true'  -d '
{
    "query" : {
        "filtered" : {
            "filter" : {
                "range" : {
                    "age" : { "gt" : 25 }
                }
            },
            "query" : {
                "match" : {
                    "user" : "leon"
                }
            }
        }
    }
}'
```

## Global search

[Match Query](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/query-dsl-match-query.html)

```go
curl 'localhost:9200/accounts/person/_search?pretty=true'  -d '
{
  "query" : { "match" : { "desc" : "軟件" }}
}'
```

```go
curl 'localhost:9200/accounts/person/_search?pretty=true&q=user:leon'
```

### `size` to limit response record

```go
curl 'localhost:9200/accounts/person/_search?pretty=true'  -d '
{
  "query" : { "match" : { "desc" : "管理" }},
  "size": 1
}'
```

### `from` to shift start index

```go
curl 'localhost:9200/accounts/person/_search?pretty=true'  -d '
{
  "query" : { "match" : { "desc" : "管理" }},
  "from": 1,
  "size": 1
}'
```

###  `or` Search

"軟件 系統" -> "軟件" or "系統"

```go
curl 'localhost:9200/accounts/person/_search?pretty=true'  -d '
{
  "query" : { "match" : { "desc" : "軟件 系統" }}
}'
```

###  `and` Search

should use [Bool Query](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/query-dsl-bool-query.html) 

must have"軟件" AND "系統"

```go
curl 'localhost:9200/accounts/person/_search?pretty=true'  -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "desc": "軟件" } },
        { "match": { "desc": "系統" } }
      ]
    }
  }
}'
```

### `phrases` Search

must be "軟件 系統"

```go
curl 'localhost:9200/accounts/person/_search?pretty=true'  -d '
{
    "query" : {
        "match_phrase" : {
            "desc" : "軟件 系統"
        }
    }
}'
```

### `aggregations` Search

count each user

```go
curl 'localhost:9200/accounts/person/_search?pretty=true'  -d '
{
  "aggs": {
    "all_user": {
      "terms": { "field": "user" }
    }
  }
}'
```

```go
//...
"aggregations" : {
    "all_user" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [ {
        "key" : "leon",
        "doc_count" : 2
      }, {
        "key" : "mark",
        "doc_count" : 1
      } ]
    }
  }
//...
```

count each user + avg age

```go
curl 'localhost:9200/accounts/person/_search?pretty=true'  -d '
{
    "aggs" : {
        "all_user" : {
            "terms" : { "field" : "user" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}'
```

```go
// ...
  "aggregations" : {
    "all_user" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [ {
        "key" : "leon",
        "doc_count" : 2,
        "avg_age" : {
          "value" : 28.0
        }
      }, {
        "key" : "leo2222n",
        "doc_count" : 1,
        "avg_age" : {
          "value" : 38.0
        }
      }, {
        "key" : "mark",
        "doc_count" : 1,
        "avg_age" : {
          "value" : 28.0
        }
      } ]
    }
  }
// ...
```

# Reference

* [elasticsearch](https://github.com/elastic/elasticsearch)
* [全文搜索引擎 Elasticsearch 入門課程](http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html)
* [Elasticsearch 權威指南](https://es.xiaoleilu.com/010_Intro/00_README.html)
