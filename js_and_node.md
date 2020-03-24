# General JS and Node notes

Nodejs is a C++ program with v8 embedded. It takes v8 and adds all the necessary features for making a server technology with javascript.
Nodejs is written in C++ - V8 is written in C++. V8 (Google, open source) converts js code into machine code based on the ECMAScript standard. V8 can be embedded into any C++ application and features can be added on top of v8.
Node also has js wrappers to C++ functionality to make it easier to use, it also has utilities being both a framework and a code library.

CommonJS modules: An agreed upon standard for how code modules should be structured.

Functions in js are first-class: you can do everything you can do with other types you can do with functions. Assign them to variables, arrays parameters etc
JS allows function-expressions (anonymous functions assigned to variables).

```js
function hi() {
  console.log('hi')
}
// first-class function
function log(hi) {
  hi()
}
log(hi)
// function-expression
var bye = function() {
  console.log('bye')
}
log(bye)
// function-expression on the fly
log(function() {
  console.log('hey)
})
```

JS has Prototypal inheritance. Objects inherit from the prototype chain. This allows adding properties to objects by changing their prototype.

JS has Immediately invoked function expressions (IIFE), allowing to protect the scope of that piece of code:

```js
(function() {}())
```

## Require and modules in node

When using require, nodejs wraps the code that is inside the module into a function expression, invokes it and returns whatever is inside the exports property of the module. That function expression accepts this parameters: exports, require, module, filename, and dirname. This is the reason for those elements to be available from inside the module when coding. Each module is just the body of a function.

Node caches in memory all requires, if exporting an object, we'll be getting a reference to the same object in different requires. Very powerfull for making db or lib wrappers with only one initialization.

Node automaticaly looks for .js and index.js files in the require dir

Functions and variables not exported in the module won't be exposed (Revealing Module Pattern)

```js
// v1
module.exports = function () {}
// v2
module.exports.foo = function () {}
// v3
function Foo() {
  this.foo = function(){}
}
module.exports = new Foo()
// v4
function Foo() {
  this.foo = function(){}
}
module.exports = Foo
// v5
function foo() {}
module.exports = { foo }

const foov1 = require('./v1')
foov1()
const foov2 = require('./v2')
foov2.foo()
const {foo} = require('./v2')
foo()
const foo = require('./v2').foo
foo()
const foov3 = require('./v3')
foov3.foo()
const Foo = require('./v4')
const foov4 = new Foo()
foov4.foo()
const foov5 = require('./v5')
foov5.foo()
const {foo} = require('./v5')
foo()
const foo = require('./v5').foo
foo()
```
