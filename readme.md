# barf - Basically A Remarkable Framework

BARF is a small and unobtrusive framework for building JSON-based web APIs on REST or GraphQL-based architectures. It is implemented such that getting started is easy and quick, but it is also flexible enough to allow for more complex use cases.

## Installation

```shell
go get github.com/opensaucerer/barf
```

## Usage

### A simple BARF REST API

```go
package main

import (
	"log"
	"net/http"

	"github.com/opensaucerer/barf"
)

func main() {

	// barf tries to be as unobtrusive as possible, so your route handlers still
	// inherit the standard http.ResponseWriter and *http.Request parameters
	barf.Get("/", func(w http.ResponseWriter, r *http.Request) {
		barf.Response(w).Status(http.StatusOK).JSON(barf.Res{
			Status:  true,
			Data:    nil,
			Message: "Hello World",
		})
	})

	// create & start server
	if err := barf.Beck(); err != nil {
		log.Fatal(err)
	}
}
```

### BARF with custom configuration

```go
package main

import (
	"log"
	"net/http"

	"github.com/opensaucerer/barf"
)

func main() {
	// create server
	allow := true
	if err := barf.Stark(barf.Augment{
		Port:     "5000",
		Logging:  &allow,  // enable request logging
		Recovery: &allow, // enable panic recovery so barf returns a 500 error instead of crashing
	}); err != nil {
		log.Fatal(err)
	}

	barf.Get("/", func(w http.ResponseWriter, r *http.Request) {
		barf.Response(w).Status(http.StatusOK).JSON(barf.Res{
			Status:  true,
			Data:    nil,
			Message: "Hello World",
		})
	})

	// start barf server
	if err := barf.Beck(); err != nil {
		log.Fatal(err)
	}
}
```

### BARF with Environment Variables

```go
package main

import (
	"log"
	"net/http"

	"github.com/opensaucerer/barf"
)

func main() {

	type Env struct {
		// Port for the server to listen on
		Port string `barfenv:"key=PORT;required=true"` // barfenv tag allows barf to load environment variables
	}

	env := new(Env) // global environment variable

	// you can use barf to dynamically load environment variables into a struct
	if err := barf.Env(env, "example/.env"); err != nil {
		log.Fatal(err)
	}

	// create server
	allow := true
	if err := barf.Stark(barf.Augment{
		Port:     env.Port,
		Logging:  &allow,
		Recovery: &allow,
	}); err != nil {
		log.Fatal(err)
	}

	barf.Get("/", func(w http.ResponseWriter, r *http.Request) {
		barf.Response(w).Status(http.StatusOK).JSON(barf.Res{
			Status:  true,
			Data:    nil,
			Message: "Hello World",
		})
	})

	// start server - create & start server
	if err := barf.Beck(); err != nil {
		log.Fatal(err)
	}
}
```

### BARF with request body, variable paths and query parameters

```go
package main

import (
	"log"
	"net/http"

	"github.com/opensaucerer/barf"
)

func main() {

	type Env struct {
		// Port for the server to listen on
		Port string `barfenv:"key=PORT;required=true"`
	}

	env := new(Env) // global environment variable

	// load environment variables
	if err := barf.Env(env, "example/.env"); err != nil {
		log.Fatal(err)
	}

	// create server
	allow := true
	if err := barf.Stark(barf.Augment{
		Port:     env.Port,
		Logging:  &allow,
		Recovery: &allow,
	}); err != nil {
		log.Fatal(err)
	}

	barf.Post("/:username", func(w http.ResponseWriter, r *http.Request) {

		var data struct {
			Name string `json:"name"`
			Age  int    `json:"age"`
		}

		err := barf.Request(r).Body().Format(&data)
		if err != nil {
			barf.Response(w).Status(http.StatusBadRequest).JSON(barf.Res{
				Status:  false,
				Data:    nil,
				Message: "Invalid request body",
			})
			return
		}

		params, _ := barf.Request(r).Params().JSON()
		query, _ := barf.Request(r).Query().JSON()

		barf.Response(w).Status(http.StatusOK).JSON(barf.Res{
			Status:  true,
			Data:    map[string]interface{}{"params": params, "query": query, "body": data},
			Message: "Hello World",
		})
	})

	// start server - create & start server
	if err := barf.Beck(); err != nil {
		log.Fatal(err)
	}
}

```

### BARF with custom middleware

```go
package main

import (
	"log"
	"net/http"

	"github.com/opensaucerer/barf"
)

func main() {
	// create server
	allow := true
	if err := barf.Stark(barf.Augment{
		Port:     "5000",
		Logging:  &allow,  // enable request logging
		Recovery: &allow, // enable panic recovery so barf returns a 500 error instead of crashing
	}); err != nil {
		log.Fatal(err)
	}

	// apply global middleware to all routes - middleware is applied in the order it is added and must be added before the call to barf.Beck()
	barf.Hippocampus().Hijack(func(h http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			log.Println("before 0")
			h.ServeHTTP(w, r)
			log.Println("after 0")
		})
	})

	// define routes
	barf.Get("/", func(w http.ResponseWriter, r *http.Request) {
		barf.Response(w).Status(http.StatusOK).JSON(barf.Res{
			Status:  true,
			Data:    nil,
			Message: "Hello World",
		})
	})

	// apply another global middleware
	barf.Hippocampus().Hijack(func(h http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			log.Println("before 1")
			h.ServeHTTP(w, r)
			log.Println("after 1")
		})
	})

	// start barf server
	if err := barf.Beck(); err != nil {
		log.Fatal(err)
	}
}
```
