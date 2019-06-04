---
layout: post
title: "Golang - GraphQL with gqlgen"
date: 2019-06-04 18:50:50 +0800
comments: true
categories: golang
---

<!-- more -->

## Setup Project

使用 Go Module  記得先執行 `export GO111MODULE=on`

```go
mkdir gqlgen-todos
cd gqlgen-todos
go mod init github.com/[username]/gqlgen-todos
```

## Define the schema

新增 `schema.graphql` 定義 [graphql 結構](https://graphql.org/learn/queries/)

```go
type Todo {
  id: ID!
  text: String!
  done: Boolean!
  user: User!
}

type User {
  id: ID!
  name: String!
}

type Query {
  todos: [Todo!]!
}

input NewTodo {
  text: String!
  userId: String!
}

type Mutation {
  createTodo(input: NewTodo!): Todo!
}
```

## Create the project skeleton

透過 `gglgen` 建立 graphql 的專案初始化

```go
go run github.com/99designs/gqlgen init
```

* `gqlgen.yml` — The gqlgen config file, knobs for controlling the generated code.
* `generated.go` — The GraphQL execution runtime, the bulk of the generated code.
* `models_gen.go` — Generated models required to build the graph. Often you will override these with your own models. Still very useful for input types.
* `resolver.go` — This is where your application code lives. generated.go will call into this to get the data the user has requested.
* `server/server.go` — This is a minimal entry point that sets up an http.Handler to the generated GraphQL server.

## Create the database models

自動生成的 Todo model 不是正確的，因為 Todo 裡面還嵌入了 User，我們希望是在使用者要求時才給予，因此要另外建立 `todo.go`

```go
// todo.go
package gqlgen_todos

type Todo struct {
	ID     string
	Text   string
	Done   bool
	UserID string
}
```

將新的 struct 路徑加到 `gqlgen.yml`

```go
// Todo 對應到 todo.go 裡的 struct name
models:
  Todo:
    model: github.com/[username]/gqlgen-todos.Todo
    
```

重新生成檔案

```go
go run github.com/99designs/gqlgen
```

## Implement the resolvers

在 `generated.go` 產生了很多個 resolvers interface，接著就要實作這些，但 `resolver.go` 這個檔案，再新增新的 method 時，要重新再產生一份

```go
rm resolver.go
go run github.com/99designs/gqlgen
```

實作 `resolver.go`

```go
package gqlgen_todos

import (
	context "context"
	"fmt"
	"math/rand"
)

type Resolver struct {
	todos []Todo
}

func (r *Resolver) Mutation() MutationResolver {
	return &mutationResolver{r}
}
func (r *Resolver) Query() QueryResolver {
	return &queryResolver{r}
}
func (r *Resolver) Todo() TodoResolver {
	return &todoResolver{r}
}

type mutationResolver struct{ *Resolver }

func (r *mutationResolver) CreateTodo(ctx context.Context, input NewTodo) (*Todo, error) {
	todo := &Todo{
		Text:   input.Text,
		ID:     fmt.Sprintf("T%d", rand.Int()),
		UserID: input.UserID,
	}
	r.todos = append(r.todos, *todo)
	return todo, nil
}

type queryResolver struct{ *Resolver }

func (r *queryResolver) Todos(ctx context.Context) ([]Todo, error) {
	return r.todos, nil
}

type todoResolver struct{ *Resolver }

func (r *todoResolver) User(ctx context.Context, obj *Todo) (*User, error) {
	return &User{ID: obj.UserID, Name: "user " + obj.UserID}, nil
}
```

# Query

```go
// open http://localhost:8080
go run server/server.go
```

```go
// create
mutation createTodo {
  createTodo(input:{text:"todo", userId:"1"}) {
    user {
      id
    }
    text
    done
  }
}

// show
query findTodos {
  	todos {
      text
      done
      user {
        name
      }
    }
}
```

## Finishing touches

At the top of our resolver.go add the following line:

```go
//go:generate go run github.com/99designs/gqlgen
```

This magic comment tells go generate what command to run when we want to regenerate our code

## Example

* [gqlgen-todos](https://github.com/mgleon08/go-packages/tree/master/020.gqlgen-todos)

# Reference

* [gqlgen getting-started](https://gqlgen.com/getting-started/)
* [Graphql](https://graphql.org/)
