# General JS and Node notes

Nodejs is a C++ program with v8 embedded. It takes v8 and adds all the necessary features for making a server technology with javascript.
Nodejs is written in C++ - V8 is written in C++. V8 (Google, open source) converts js code into machine code based on the ECMAScript standard. V8 can be embedded into any C++ application and features can be added on top of v8.
Node also has js wrappers to C++ functionality to make it easier to use, it also has utilities being both a framework and a code library.

Node adds:

- modules
- file and data base management
- communication over the internet
- ability to accept requests and send response
- a way to deal with time consuming tasks

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
(function() {})();
(function() {}());
(() => {})(); // ES6
```

## Require and modules in node

When using require, nodejs wraps the code that is inside the module into a function expression, invokes it and returns whatever is inside the exports property of the module. That function expression accepts this parameters: exports, require, module, filename, and dirname. This is the reason for those elements to be available from inside the module when coding. Each module is just the body of a function. ES6 supports modules with `export function...` and `import * from ...`.

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

exports can be used in modules if we mutate it and not replace it (equal). If we equal exports to an object we break the reference and it won't be returned when requiring that module. It is preferable to user `module.exports`

Native modules are also imported with require. Node will first look for native and installed modules when importing.

```js
require('crypto')
require('util')
```

## Events and inheritance

System events (libuv, c library) come from the c++ core and represent events that are not part of JS, closer to the machine. Custom events are JS related and belong to the Event Emitter. Sometimes events from libuv trigger custom JS events.

The node event emitter basically pushes listeners to the event prop array and executes them when the event is emitted.

Apply or call a function changes the scope of the function. Overwrite 'this'.

```js
obj.doSomething()
obj.doSomething.call({}, param) // The parameter of call overwrites 'this'
obj.doSomething.apply({}, []) // The parameter of call overwrites 'this'
```

Overwriting this helps when dealing with object inheritance.

```js
function Foo() {
  this.foo = 'foo'
}
function Bar() {
  Foo.call(this) // making .call here will make this.foo available here
  this.bar = 'bar'
}
const barObj = util.inherits(Bar, Foo) // inherits comes from util native module

```

ES6 introduces classes to improve OOP in js, it is just syntactic sugar that still uses prototypical inheritance in js.

```js
class Foo {
  constructor() {
    this.foo = 'foo'
  }
}
class Bar extends Foo {
  constructor() {
    super() // super will make this.foo available here
    this.bar = 'bar'
  }
}
```

## Asynchrony and synchrony

JS is synchronous, one line at a time. Asynchrony in node allows other modules like libuv to process tasks from the event loop. When Node.js needs to perform an I/O operation instead of blocking the thread and wasting CPU cycles waiting, Node.js will resume the operations when the response comes back from libuv.

## Buffers and Streams

Buffers holds binary data. It is a fundamental part of node, they are global so no need to require anything. It defaults to utf8 encoding. Buffers can be converted to strings, jsons... It behaves like an array to access its data. ES6 can deal with binary data with typed arrays.

Node uses buffers for loading contents of files through the fs native module. Carefull with readFileSync because it blocks the loop, it is recommended to use it just for loading config data at the init of the application. readFile accepts a callback and won't block the event loop, it's an error-first callback. 

Error-first callback -> callback that returns an error as the first paramenter, or null if success.

The heap of v8 can be affected when reading large files and not using streams. Streams inherit EventEmitter so the EventEmitter methods are available in all streams. Streams can be readeable, writable, duplex... Streams are abstracts so it should be inherited and not used directly. fs provides createReadStream and createWriteStream. 

Readable streams have a pipe function to pipe data to a given destination.
