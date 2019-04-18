---
layout: post
title: "Golang - MongoDB"
date: 2019-04-10 21:30:30 +0800
comments: true
categories: golang mongodb
---

<!-- more -->

# Install the MongoDB Go Driver

```go
go go get github.com/mongodb/mongo-go-driver
```

或使用 go module

```go
go mode init <current package name>
```

# Connect to MongoDB using the Go Driver

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/mongodb/mongo-go-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

func main() {
	client, err := mongo.Connect(context.Background(), options.Client().ApplyURI("mongodb://localhost:27017"))

	if err != nil {
		log.Fatal(err)
	}

	// Check the connection
	err = client.Ping(context.Background(), nil)

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println("Connected to MongoDB!")

	// handle for the trainers collection in the test database
	// collection := client.Database("test").Collection("trainers")

	// Close the connection
	err = client.Disconnect(context.TODO())

	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Connection to MongoDB closed.")
}
```

* `mongo.Connect` Connect
* `client.Ping` Check the connection
* `client.Disconnect` Close the connection

# BSON (Binary Serialized Document Format, Binary-encoded JSON)

* 一種二進制形式的存儲格式，採用了類似於 C 語言結構體的名稱
* 對表示方法支援內嵌的文件物件和陣列物件
* 具有輕量性、可遍歷性、高效性的特點，可以有效描述非結構化數據和結構化數據
* schema-less的存儲形式
* 優點是靈活性高，但它的缺點是空間利用率不是很理想
* 不像一般 json 用簡單的 string 和 number，而是可以給予 type (int, long, date, floating point, and decimal128)，可以更方便的去做比較排許等等的


### 在 Go 中提供了兩種 BSON 的資料格式 `D types` the `Raw types`

`D type`

> The D family of types is used to concisely build BSON objects using native Go types. This can be particularly useful for constructing commands passed to MongoDB. The D family consists of four types:

* `D`: A BSON document. This type should be used in situations where order matters, such as MongoDB commands.
* `M`: An unordered map. It is the same as D, except it does not preserve order.
* `A`: A BSON array.
* `E`: A single element inside a D.

Example

```go
// D type filter to find where the name field matches either Alice or Bob 
bson.D\{\{
    "name", 
    bson.D\{\{
        "$in", 
        bson.A{"Alice", "Bob"}
    }}
}}
```

`Raw`

> The Raw family of types is used for validating a slice of bytes. You can also retrieve single elements from Raw types using a [Lookup](https://godoc.org/go.mongodb.org/mongo-driver/bson#Raw.Lookup). This is useful if you don't want the overhead of having to unmarshall the BSON into another type. This tutorial will just use the D family of types.

# CRUD Operations

```go
type Trainer struct {
	Name string
	Age  int
	City string
}
```

### C - Insert documents

```go
ash := Trainer{"Ash", 10, "Pallet Town"}
misty := Trainer{"Misty", 10, "Cerulean City"}
brock := Trainer{"Brock", 15, "Pewter City"}

// insert one
insertResult, err := collection.InsertOne(context.TODO(), ash)
if err != nil {
  log.Fatal(err)
}
fmt.Println("Inserted a single document: ", insertResult.InsertedID)

// insert many
trainers := []interface{}{misty, brock}
insertManyResult, err := collection.InsertMany(context.TODO(), trainers)
if err != nil {
  log.Fatal(err)
}
fmt.Println("Inserted multiple documents: ", insertManyResult.InsertedIDs)
```

### U - Update documents

需要透過 `bson.D` types 去找到 match 的 data 再做更新

* [Field Update Operators](https://docs.mongodb.com/manual/reference/operator/update-field/)

```go
// 找出 name ＝ "Ash"
filter := bson.D{{"name", "Ash"}}

// 將 age - 1
update := bson.D{
    {"$inc", bson.D{
        {"age", 1},
    }},
}

updateResult, err := collection.UpdateOne(context.TODO(), filter, update)
if err != nil {
    log.Fatal(err)
}

fmt.Printf("Matched %v documents and updated %v documents.\n", updateResult.MatchedCount, updateResult.ModifiedCount)
```

* `$inc` - Increments the value of the field by the specified amount.
* `updateResult.MatchedCount` - The number of documents that matched the filter.
*  `updateResult.ModifiedCount` - The number of documents that were modified.

### R - Find documents

`Find a document` use `collection.FindOne()`

> To find a document, you will need a filter document as well as a pointer to a value into which the result can be decoded

```go
// create a value into which the result can be decoded
var result Trainer

// 取出的值必須要 Decode
err = collection.FindOne(context.TODO(), filter).Decode(&result)
if err != nil {
    log.Fatal(err)
}

fmt.Printf("Found a single document: %+v\n", result)
```

`Find multiple documents` use `collection.Find()`

> This method returns a Cursor. A Cursor provides a stream of documents through which you can iterate and decode one at a time. Once a Cursor has been exhausted, you should close the Cursor.
> 
> Here you'll also set some options on the operation using the options package. Specifically, you'll set a limit so only 2 documents are returned.

```go
// Pass these options to the Find method
findOptions := options.Find()
findOptions.SetLimit(2)

// Here's an array in which you can store the decoded documents
var results []*Trainer

// Passing nil as the filter matches all documents in the collection
// 範例原本寫 nil 但是有問題所以改成 bson.D{}
cur, err := collection.Find(context.TODO(), bson.D{}, findOptions)
if err != nil {
    log.Fatal(err)
}

// Finding multiple documents returns a cursor
// Iterating through the cursor allows us to decode documents one at a time
for cur.Next(context.TODO()) {

    // create a value into which the single document can be decoded
    var elem Trainer
    err := cur.Decode(&elem)
    if err != nil {
        log.Fatal(err)
    }
    results = append(results, &elem)
}

if err := cur.Err(); err != nil {
    log.Fatal(err)
}

// Close the cursor once finished
cur.Close(context.TODO())

fmt.Printf("Found multiple documents (array of pointers): %+v\n", results)

for index, result := range results {
    fmt.Printf("%d: %+v \n", index, result)
}
```

### D - Delete Documents

> delete documents using `collection.DeleteOne()` or `collection.DeleteMany()`. 
> 
> Here you pass nil as the filter argument, which will match all documents in the collection. You could also use [collection.Drop()](https://godoc.org/github.com/mongodb/mongo-go-driver/mongo#Collection.Drop) to delete an entire collection.

```go
// DeleteOne
deleteOneResult, err := collection.DeleteOne(context.TODO(), bson.M{})
if err != nil {
  log.Fatal(err)
}
fmt.Printf("Deleted %v documents in the trainers collection\n", deleteOneResult.DeletedCount)

// DeleteMany
deleteResult, err := collection.DeleteMany(context.TODO(), bson.M{})
// DELETA name = Misty && age > 10
// deleteResult, err := collection.DeleteMany(context.TODO(), bson.M{"name": "Misty", "age": bson.M{"$gte": 10}})
// deleteResult, err := collection.DeleteMany(context.TODO(), bson.D\{\{"name", "Misty"}, {"age", bson.M{"$gte": 10}}})
if err != nil {
  log.Fatal(err)
}
fmt.Printf("Deleted %v documents in the trainers collection\n", deleteResult.DeletedCount)
```

# Example

* [016.mongodb](https://github.com/mgleon08/go-packages/tree/master/017.mongodb)

Reference

* [documentation_examples](https://github.com/mongodb/mongo-go-driver/tree/master/examples/documentation_examples)
* [godoc mongo-driver](https://godoc.org/go.mongodb.org/mongo-driver)
* [BSON Operators](https://docs.mongodb.com/manual/reference/operator/)
* [BSON](http://bsonspec.org/)
* [BSON的介紹及BSON與JSON的區別](https://blog.csdn.net/m0_38110132/article/details/77716792)
* [MongoDB Go Driver Tutorial](https://www.mongodb.com/blog/post/mongodb-go-driver-tutorial)
* [mongodb官方的golang驅動基礎使用課程分享](https://www.jianshu.com/p/0344a21e8040?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)
* [MongoDB官方推出的Go驅動庫“mongo-go-driver”快速課程](https://zh.shellman.me/articles/mongo-go-driver-demo/)
* [MongoDB 基礎入門教學：MongoDB Shell 篇](https://blog.gtwang.org/programming/getting-started-with-mongodb-shell-1/)

