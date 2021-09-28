---
title: Node
date: 2021-07-12 12:36:36
categories: 
- programming languages
tags: 
- javascript
- node
---
## Key Usage/Concepts

### `var` vs `let` vs `const`
```
var:
  function scoped
  undefined when accessing a variable before it's declared

let:
  block scoped
  ReferenceError when accessing a variable before it's declared

const:
  block scoped
  ReferenceError when accessing a variable before it's declared
  can't be reassigned
```

### Lambda
```javascript
Basic syntax:
param => expression
param => {
  let a = 1;
  return a + param;
}
(param1, paramN) => {
   let a = 1;
   return a + param1 + paramN;
}

Advanced syntax:
params => ({foo: "a"}) // returning the object {foo: "a"}
(a=400, b=20, c) => expression

Destructuring:
([a, b] = [10, 20]) => a + b;  // result is 30
({ a, b } = { a: 10, b: 20 }) => a + b; // result is 30
```

#### `apply` vs `bind` vs `call`
![apply vs bind vs call][1]
-   _Call_  invokes the function and allows you to pass in arguments one by one.
-   _Apply_  invokes the function and allows you to pass in arguments as an array.
-   _Bind_  returns a new function, allowing you to pass in a this array and any number of arguments.

```javascript
call: 
var person1 = {firstName: 'Jon', lastName: 'Kuperman'};
var person2 = {firstName: 'Kelly', lastName: 'King'};

function say(greeting) {
    console.log(greeting + ' ' + this.firstName + ' ' + this.lastName);
}

say.call(person1, 'Hello'); // Hello Jon Kuperman
say.call(person2, 'Hello'); // Hello Kelly King

apply:
var person1 = {firstName: 'Jon', lastName: 'Kuperman'};
var person2 = {firstName: 'Kelly', lastName: 'King'};

function say(greeting) {
    console.log(greeting + ' ' + this.firstName + ' ' + this.lastName);
}

say.apply(person1, ['Hello']); // Hello Jon Kuperman
say.apply(person2, ['Hello']); // Hello Kelly King

bind:
var person1 = {firstName: 'Jon', lastName: 'Kuperman'};
var person2 = {firstName: 'Kelly', lastName: 'King'};

function say() {
    console.log('Hello ' + this.firstName + ' ' + this.lastName);
}

var sayHelloJon = say.bind(person1);
var sayHelloKelly = say.bind(person2);

sayHelloJon(); // Hello Jon Kuperman
sayHelloKelly(); // Hello Kelly King
```
`call` and `apply` are pretty interchangeable. Just decide whether it’s easier to send in an array or a comma separated list of arguments.

`bind` is a bit different. It returns a new function. `call` and `apply` execute the current function immediately.

### [Comparison Operators][2]

## TypeScript

### `instanceOf` vs `typeof`

```javascript
Use  instanceof for custom types:
var ClassFirst = function () {};
var ClassSecond = function () {};
var instance = new ClassFirst();
typeof instance; // object
typeof instance == 'ClassFirst'; // false
instance instanceof Object; // true
instance instanceof ClassFirst; // true
instance instanceof ClassSecond; // false 

Use typeof for simple built in types:
'example string' instanceof String; // false
typeof 'example string' == 'string'; // true
'example string' instanceof Object; // false
typeof 'example string' == 'object'; // false
true instanceof Boolean; // false
typeof true == 'boolean'; // true
99.99 instanceof Number; // false
typeof 99.99 == 'number'; // true
function() {} instanceof Function; // true
typeof function() {} == 'function'; // true

Use instanceof for complex built in types:
/regularexpression/ instanceof RegExp; // true
typeof /regularexpression/; // object
[] instanceof Array; // true
typeof []; //object
{} instanceof Object; // true
typeof {}; // object

```

## Reference

1. [JavaScript Introduction][3]
2. [TypeScript Introduction][4]

[1]:https://raw.githubusercontent.com/eziceice/blog/master/node.js/applyvsbindvscall.jpeg
[2]:https://www.w3schools.com/js/js_comparisons.asp
[3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Introduction#where_to_find_javascript_information
[4]: https://www.typescriptlang.org/docs/handbook/2/basic-types.html
