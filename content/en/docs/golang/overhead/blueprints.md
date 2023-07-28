---
title: "Blueprints"
linkTitle: "Blueprints"
weight: 200
description: >
  Implementation details for application strategies.
---

## Web applications

Web applications have an `application` struct that contains the following:
- loggers
- models (with db connections)
- templateCache
- (optional) form decoder
- (optional) session manager

```shell
project/
├── cmd
│   └── web
│       ├── context.go
│       ├── handlers.go
│       ├── handlers_test.go
│       ├── helpers.go
│       ├── main.go
│       ├── middleware.go
│       ├── middleware_test.go
│       ├── routes.go
│       ├── templates.go
│       └── testutils_test.go
├── go.mod
├── go.sum
├── internal
│   ├── assert
│   │   └── assert.go
│   ├── models
│   │   ├── errors.go
│   │   ├── mocks
│   │   │   ├── snippets.go
│   │   │   └── users.go
│   │   ├── snippets.go
│   │   ├── testdata
│   │   │   ├── setup.sql
│   │   │   └── teardown.sql
│   │   ├── testutils_test.go
│   │   ├── users.go
│   │   └── users_test.go
│   └── validator
│       └── validator.go
├── README.md
├── tls
│   ├── cert.pem
│   └── key.pem
└── ui
    ├── html
    │   ├── base.tmpl.html
    │   ├── pages
    │   │   ├── create.tmpl.html
    │   │   ├── home.tmpl.html
    │   │   ├── login.tmpl.html
    │   │   ├── signup.tmpl.html
    │   │   └── view.tmpl.html
    │   └── partials
    │       └── nav.tmpl.html
    └── static
        ├── css
        │   └── main.css
        ├── img
        │   ├── favicon.ico
        │   └── logo.png
        └── js
            └── main.js


```
### `/cmd`

Holds executables

#### `/web`

Holds executables for the web application:
  - `context.go`
  - `handlers.go`
  - `helpers.go`
  - `main.go`
  - `middleware.go`
  - `routes.go`
  - `templates.go`

### `/internal`

#### `/models`

#### `/validator`

### `/tls`

### `/ui`

#### `/html`

#### `/static`

### Steps

1. Create your templates, render method, handlers for the templates.
2. Set up your router.
3. Add error and info loggers to main, and the application.
4. Set up `serverError` helper method that logs the trace to the errorLog and sends an `http.Error()` message.
5. Set up `clientError` helper that validates that form submissions use the correct HTTP verb.
6. Set up `notFound` helper to manage client requests for pages that do not exist.
7. Add middleware logger, security header, and panic recovery. Use `github.com/justinas/alice` to simplify routing. Add them as methods to the application so they can access application dependencies, like loggers.


## Service blueprint

Services do not require an `application` struct to manage user details.

```shell
shortner/
├── cmd
│   └── servicename
│       └── main.go or service.go
├── internal
│   └── httpio
│       ├── handler.go
│       └── httpio.go
├── linkit
│   └── errors.go
└── short
    ├── server.go
    ├── server_test.go
    └── business-logic.go

```


## Template data

## Hashing and storing passwords