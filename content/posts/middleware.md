---
title: "Middleware"
date: 2021-06-27T14:47:47+02:00
draft: true
---

Some functionality is shared between handlers. Authentication, logging and rate limiting are some that comes to mind. Wouldn't it be nice to not have to add that functionality to each new handler? With middlewares you can! Middlewares intercept an HTTP request and enables one to do stuff before the original handler function is excecuted.

By satisifying the `http.Handler` interface we can chain as many middlewares as we'd like.

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

func loggerMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		fmt.Printf("%s %s %s\n", time.Now(), r.Method, r.URL)
		next.ServeHTTP(w, r)
	})
}

func answerHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "42")
}

func main() {
	mux := http.NewServeMux()
	mux.Handle("/answer", loggerMiddleware(http.HandlerFunc(answerHandler)))
	log.Fatal(http.ListenAndServe(":8080", mux))
}
```
