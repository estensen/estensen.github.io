---
title: "Creating a Go API"
date: 2021-06-27T11:04:11+02:00
draft: true
---

Often creating a service starts with defining an API. HTTP is the most common way of defining a way for a client to interact with service data. A browser can issue a request, ask the server to do something, by sending an HTTP method and a resource path. For example `GET /answer`. The server then has to match this request to its server logic. This is what the request router is for.


```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func answerHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Println("Client asked for answer")
	fmt.Fprintf(w, "42")
}

func main() {
	http.HandleFunc("/answer", answerHandler)
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

`HandleFunc` registers the handler function `answerHandler` on the router.

`ListenAndServe` listenes for TCP connections and calls `Serve` with a handler to handle incoming connections. `Serve` creates a goroutine, a lightweight thread, for each requests, which enables the API to handle lots of concurrent requests. The second argument for `ListenAndServe` defines the request multiplexer, used to match the request to the list of registered routes. When this is `nil` the `DefaultServeMux` is used.
