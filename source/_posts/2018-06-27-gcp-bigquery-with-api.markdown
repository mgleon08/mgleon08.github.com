---
layout: post
title: "GCP BigQuery with API"
date: 2018-06-27 21:18:35 +0800
comments: true
categories: rails other
---

最近剛好碰到 BigQuery，可以直接透過 API 帶 sql 指令去拉資料!

<!-- more -->

### 安裝

```ruby
# Gemfile
gem 'google-cloud-bigquery'
```

```ruby
bundle
```

裝 [安 gcp SDK](https://cloud.google.com/sdk/docs/downloads-interactive)

```ruby
# 安裝
curl https://sdk.cloud.google.com | bash
# Restart your shell
exec -l $SHELL
# 記得在 google 那邊要先開權限，才可以指定到自己要的 project
gcloud init
# auth，但最好的話是要申請一個 憑證，並設定在環境變數(environmentvalue)
gcloud auth application-default login
# 取得憑證
gcloud auth application-default print-access-token
```

### 執行

```ruby
require "google/cloud/bigquery"

# This uses Application Default Credentials to authenticate.
# @see https://cloud.google.com/bigquery/docs/authentication/getting-started
bigquery = Google::Cloud::Bigquery.new(project: "project_id")

sql     = "SELECT " +
          "CONCAT('https://stackoverflow.com/questions/', " +
          "       CAST(id as STRING)) as url, view_count " +
          "FROM `bigquery-public-data.stackoverflow.posts_questions` " +
          "WHERE tags like '%google-bigquery%' " +
          "ORDER BY view_count DESC LIMIT 10"
results = bigquery.query sql

results.each do |row|
  puts "#{row[:url]}: #{row[:view_count]} views"
end
```

### 問題

在執行上有遇到一些問題，ruby 用 query 第一次去打的時候，回來的值卻會是空 Array，第二次就會有值

可以看到下面 `job_complete` 也是 `false`，第二次打就會是 true

```ruby
bigquery.query(sql)

# Sending HTTP post https://www.googleapis.com/bigquery/v2/projects/project_id/queries?
# 200
# #<Hurley::Response POST https://www.googleapis.com/bigquery/v2/projects/project_id/queries == 200 (184 bytes) 11197ms>
# Success - #<Google::Apis::BigqueryV2::QueryResponse:0x007fa1021033f8
#  @job_complete=false,
#  @job_reference=
#   #<Google::Apis::BigqueryV2::JobReference:0x007fa102102318
#    @job_id="job_0HlGAo3KB4WrAGz20MypIHluup9B",
#    @project_id="project_id">,
#  @kind="bigquery#queryResponse">
#  => []
```

### 原因

Google 回覆

`job_complete=false` 代表這個 bigquery job 還沒跑完.

在 bigquery 下 query 的時候, 執行 query 的 API call 會有一個 timeout, 當 API call 執行的時間超過這個 timeout 但是 query 還沒跑完的時候, API call 會 return HTTP code 200, 但是 query result 是空的, 並且 `job_complete = false.` 

詳細可以參考文件[1]一開頭 "Runs a BigQuery SQL query and returns results if the query completes within a specified timeout." 以及同一份文件下方的 timeoutMs 以及 jobComplete 這些參數.

如前所述, 如果收到 job_complete = false 的 HTTP code 200 response, 表示 job 還在執行. 這時候的標準做法是去 poll GetQueryResults 這個 API (參考文件[2]) 直到 job_complete = true 再拿取結果.

* [1. query](https://cloud.google.com/bigquery/docs/reference/rest/v2/jobs/query)
* [2. getQueryResults](https://cloud.google.com/bigquery/docs/reference/rest/v2/jobs/getQueryResults)

### 解決方式

在 new 的時候，新增 timeout 時間(也可以設定 retries)

```ruby
Google::Cloud::Bigquery.new(project: project_id, timeout: 120, retries: 10)
```

> 但實際測試加上 `timeout` 卻好像沒有用..

在試的時候，有用另外的方式去解決，自己去處理 job 就可以確保 job 跑完，再拿資料回來

```ruby
# 先設定 job
bigquery.query_job(sql)
# 再讓它去執行 wait_until_done!
job.wait_until_done!
# 最後再將結果回傳，這樣就可以確保第一次可以拉到值
job.query_results
```

但去看原始碼，實際上也是做一樣的動作..

> 因該是內建的 `query` 有設定 10 秒就會 response，因此改直接去執行裡面的動作，就沒有這個限制 

```ruby
def query query, params: nil, external: nil, max: nil, cache: true,
          standard_sql: nil, legacy_sql: nil, &block
  job = query_job query, params: params, external: external,
                  cache: cache, standard_sql: standard_sql,
                  legacy_sql: legacy_sql, &block
  job.wait_until_done!
  ensure_job_succeeded! job
  job.data max: max
end
```

參考文件

* [Create A Simple Application With the API](https://cloud.google.com/bigquery/create-simple-app-api)
* [google-cloud-bigquery](https://github.com/GoogleCloudPlatform/google-cloud-ruby/tree/master/google-cloud-bigquery)
* [安裝 gcp SDK](https://cloud.google.com/sdk/docs/downloads-interactive)
* [Command Line](https://cloud.google.com/bigquery/docs/bq-command-line-tool)
* [Auth](https://cloud.google.com/docs/authentication/production)
* [在 GOOGLE CLOUD PLATFORM 上執行 RUBY](https://cloud.google.com/ruby/)

Auth

* [How to query BigQuery programmatically from Python without end-user interaction?](https://stackoverflow.com/questions/13212991/how-to-query-bigquery-programmatically-from-python-without-end-user-interaction)
* [gcloud auth application-default login](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login)
