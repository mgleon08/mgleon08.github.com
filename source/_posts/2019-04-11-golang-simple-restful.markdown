---
layout: post
title: "Golang - Simple RESTful"
date: 2019-04-11 16:27:10 +0800
comments: true
categories: golang
---

<!-- more -->

Practice RESTful with golang

```go
GetUsers     GET     /users
GetUser      GET     /users/{id}
CreateUser   POST    /users
UpdateUser   UPDATE  /users/{id}
DeleteUser   DELETE  /users/{id}
```

Use [gorilla/mux](https://github.com/gorilla/mux) package

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"strconv"

	"github.com/gorilla/mux"
)

type User struct {
	ID        int    `json:"id,omitempty"`
	Firstname string `json:"firstname,omitempty"`
	Lastname  string `json:"lastname,omitempty"`
	Info      `json:"info,omitempty"`
}
type Info struct {
	City  string `json:"city,omitempty"`
	Phone int    `json:"phone,omitempty"`
}

var users []User

func GetUsers(w http.ResponseWriter, r *http.Request) {
	json.NewEncoder(w).Encode(users)
}

func GetUser(w http.ResponseWriter, r *http.Request) {
	id := convertParams(r)
	for _, u := range users {
		if u.ID == id {
			json.NewEncoder(w).Encode(u)
			return
		}
	}
	json.NewEncoder(w).Encode("User not found")
}

func CreateUser(w http.ResponseWriter, r *http.Request) {
	var user User
	_ = json.NewDecoder(r.Body).Decode(&user)
	users = append(users, user)
	json.NewEncoder(w).Encode(user)
}

func UpdateUser(w http.ResponseWriter, r *http.Request) {
	var user User
	id := convertParams(r)
	_ = json.NewDecoder(r.Body).Decode(&user)
	for i, u := range users {
		if u.ID == id {
			users[i] = user
			json.NewEncoder(w).Encode(user)
			break
		}
	}
}

func DeleteUser(w http.ResponseWriter, r *http.Request) {
	id := convertParams(r)
	for i, u := range users {
		if u.ID == id {
			copy(users[i:], users[i+1:])
			users = users[:len(users)-1]
			break
		}
	}
	json.NewEncoder(w).Encode(users)
}

func convertParams(r *http.Request) int {
	params := mux.Vars(r)
	id, err := strconv.Atoi(params["id"])
	if err != nil {
		log.Fatalf("err: %v", err)
	}
	return id
}

func main() {
	router := mux.NewRouter()
	user1 := User{ID: 1, Firstname: "hello", Lastname: "World", Info: Info{City: "Taipei", Phone: 123}}
	user2 := User{ID: 2, Firstname: "hello", Lastname: "World", Info: Info{City: "Taipei", Phone: 456}}
	users = append(users, user1, user2)
	router.HandleFunc("/users", GetUsers).Methods("GET")
	router.HandleFunc("/users/{id}", GetUser).Methods("GET")
	router.HandleFunc("/users", CreateUser).Methods("POST")
	router.HandleFunc("/users/{id}", UpdateUser).Methods("PUT")
	router.HandleFunc("/users/{id}", DeleteUser).Methods("DELETE")
	fmt.Println("Starting server on port 3000...")
	log.Fatal(http.ListenAndServe(":3000", router))
}
```

params 

```go
{
  "id": 3,
  "firstname": "hello",
  "lastname": "World",
  "info": {
    "city": "Taipei",
    "phone": 123
  }
}
```

run

```go
go run main.go
// http://localhost:3000/users
```

# Code

* [simple_restful](https://github.com/mgleon08/go-packages/blob/master/018.simple_restful/main.go)


# With mongodb

* [restful_mongodb](https://github.com/mgleon08/go-packages/tree/master/019.restful_mongodb)

Reference

* [How to build a RESTful API in Go for phonebook app](https://medium.com/@petrousov/how-to-build-a-restful-api-in-go-for-phonebook-app-d55f7234a10)
* [Implementing CRUD operations with Go and MongoDB](https://medium.com/@petrousov/implementing-crud-operations-with-go-and-mongodb-cf622f2379c4)
* [gorilla/mux](https://github.com/gorilla/mux)
* [2 ways to delete an element from a slice](https://yourbasic.org/golang/delete-element-slice/)
