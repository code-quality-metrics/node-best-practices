# Course

grunt, js task runner. Runs linter. Add a Gruntfile.js to add the config and the tasks.

frameworks
express, lightweigh, well documented, extensible easy to use.
koa, geddy, sails...

## 2

### Organization

Don't overthink it at the start. Don't create one directory with all the code but multiple directories for each module. (membership, sales...)

Each module has lib,  models, tests

## 3

constructor pattern

module pattern wrap functionality in an object. Is what CommonJS is based on.

```js
const MyModule = {
  id: 1,
  getId: () => MyModule.id
}
```

revealing module pattern

```js
const MyModule = () => ({
  const id = 1
  const getId = () => id
  return {
    findId: getId,
  }
})()
```

prototype

```js
const MyModule = function() {
  const id = 1
}
MyModule.prototype.getId = function() {
  return this.id
}
```

rethinkdb (mongodb done right)

Using event emitter to solve the pyramid of doom.
make Registration inherit event emitter `util.inherits(Registration, Emitter)`. Create functions for each feature and then controll the process with events. Each function will emit a certan event when completed and be triggered after a certain event.
