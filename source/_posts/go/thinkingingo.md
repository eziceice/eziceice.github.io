---
title: Thinking in Go
date: 2021-10-01 15:07:43
tags: go
categories:
- programming languages
---

## Basic

### Common Commands

- Go executes `init` functions automatically at program startup, after global variables have been initialised
- `go build` compiles the code into an executable file
- `go list -f '{{.Target}}'` show the GOPATH

### Flow Control

- Go loop doesn't need `()` same as `if`
- `defer` statement defers the execution of a function until the surrounding function returns. The arguments are evaluated immediately, but the function call is not executed until the surrounding function returns.
- Deferred function calls are pushed onto a stack. When a function returns, its deferred calls are executed in **last-in-first-out** order.

### More Types

- A slice does not store any data, it just describes a section of an underlying array. Changing the elements of a slice modifies the corresponding elements of its underlying array.
- Functions are values too, which can be passed around just like other values. 
