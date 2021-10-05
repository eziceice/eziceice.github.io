---
title: Thinking in Go
date: 2021-10-01 15:07:43
updated: 2021-10-05 15:07:43
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

### Methods

- Go does not have classes. However, you can define methods on types.
- A method is a function with a special  _receiver_  argument.
- The receiver appears in its own argument list between the  `func` keyword and the method name.
- You can only declare a method with a receiver whose type is defined in the same package as the method. You cannot declare a method with a receiver whose type is defined in another package
- With a value receiver, method operates on a copy of the original type value, if the original value needs to be changed then must use pointer.
- Functions with a pointer argument must take a pointer, while methods with pointer receivers take either a value or a pointer as the receiver when they are called.
```
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK
```
- In general, all methods on a given type should have either value or pointer receivers, but not a mixture of both
    - The first is so that the method can modify the value that its receiver points to.
    - The second is to **avoid copying the value on each method call**. This can be more efficient if the receiver is a large struct, for example.
- A type implements an interface by implementing its methods. **There is no explicit declaration of intent, no "implements" keyword**. Implicit interfaces decouple the definition of an interface from its implementation, which could then appear in any package without prearrangement.
- An interface value holds a value of a specific underlying concrete type.
- If the concrete value inside the interface itself is nil, the method will be called with a nil receiver.
- Calling a method on a nil interface is a run-time error because there is no type inside the interface tuple to indicate which _concrete_ method to call.
- Empty interfaces are used by code that handles values of unknown type
- A nil `error` denotes success; a non-nil `error` denotes failure.
