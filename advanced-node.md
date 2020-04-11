# Advanced node js

## Node != javascript

### V8 and libuv

Default node VM. Node can use also the Chakra core.

libuv abstracts non blocking io operations (file system, network sockets). Libuv is the one providing node with the event loop.

### Node repl

Node.js comes with virtual environment called REPL (aka Node shell). REPL stands for Read-Eval-Print-Loop. The default repl can be imported as a module and by calling the `start` function it can be extended.

- --harmony flag - for using staged features of the V8. There are specific harmony flags for each in progress feature of V8.

- -p evaluates and prints the results of a string. `node -p "os.cpus()"` prints the processors node sees.

- node cli provides autocomplete buy tabbing on any object or variable.

- v8.getHeapStatistics() prints heap data to the console

- underscore '_' prints the last computed value

### Global, process, buffer

`global` is the only true global variable in node. We should avoid using it.

The process object is a brigde between the node app and the running environment. It is an instance of Event Emitter. So we can listen for example to the process.on('exit') event executed when the event loop has nothing else to process or `process.exit` is manually executed. Be careful when listening to the `process.uncaughtException` event since that will prevent node from exiting the process unless you manually execute `process.exit` on that listener.

- `process.versions` gets all the node modules versions.

- `process.env` provides the user environment. If we modify process.env we are not modifying the original user environment. process.env should only be accessed through a centralized config file.

`Buffer` is a chunk of memory outside of the v8 heap. When reading we have to specify an encoding. Buffers are created with alloc, allocUnsafe, from. allocUnsafe won't fill the memory space and that means there can be some old data there. Buffers are useful for reading files or any binary data access.

When converting a buffer to a string always use StringDecoder since it handles multi byte characters and inclomplete multi byte characters.

### modularity in node

`require.resolve` does not load the module, it can be used to check if a package is available or not.

node caches every required module for any other requests. The cache is managed in an array inside require `require.cache`. Automaticaly gets index.js inside directories.

module.exports contains what will be exported to other modules.

require can also get jsons (JSON.parse) files and c++ addons compiled into .node files (process.dlopen). `node-gyp` is used for compiling files.

We can use the exports object to export properties but we cannot replace the exports object directly, for that we use the module.exports. Exports is a variable reference to module.exports.

Scoping modules in node is done by wrapping the module in a function with five arguments. exports, require, module, __filename, __dirname. It keeps the top level variables scoped to that module. exports is a reference to module.exports

## Concurrency and event loop

The event loop. Slow i/o operations are handled by events and callbacks. i/o operations can be memory disk network or other processes. Disk and netwokr resources are the most expensive operations. Node minimizes waiting time. Node does not use threads, it's single threaded.

The event loop picks events from the event queue and pushes their callbacks to the call stack. It starts when a script is started and exited when no more callbacks or process.exit is called.

V8 has a call stack and a heap. The heap is where objects are stored in memory. Both are part of the runtime engine not node itself. The loop is provided by the libuv.

The call stack is a stack of functions. Only one stack. Only one function executed at a time. When executing a script every time we step into a function it is pushed into the stack and everytime we return from a function it gets popped out of the stack. It is printed on errors.

Queue is a queue of events to be executed. When executing we move the callback to the stack. settimeout is provided by node and node executes a timer out of the runtime freeing the call stack to process more items. When the timer finishes node sends the callback to the event queue. Then the event loop will take events form the queue and add them to the stack when the stack is empty.

When the timer delay is 0, it works in the same way as setImmediate. The callbacks will be executed after we are done with the stack. setImmediate can take precedende over settimeout with 0. process.nexttick is not part of the event loop, callbacks are processed after the current operation completes and before the event loop continues. process.nexttick can be used when returning a custom error in a async function asyncronussly.

## Event driven architecture

Event emitter objects provide the functions emit and on to emit or listen to events. The on method adds listener functions.

## Node for networking

### net server

```js
const server = require('net').createServer();
```

Now the server is an instance of event emitter and we can listen `on` connections. There, we get a socket. The socket implements a duplex stream interface so that we can write to clients and read from clients. The reading handlers will get the data as buffers.

```js
server.on('connection', socket => {
  console.log('Client connected');
  socket.write('Welcome new client!\n');

  socket.on('data', data => {
    console.log('data is:', data);
    socket.write('data is: ');
    socket.write(data);
  });

  socket.on('end', () => {
    console.log('Client disconnected');
  });
});
```

We also have to listen to a port to run the server

```js
server.listen(8000, () => console.log('Server bound'));
```

For making a basic chat we can read data from the sockets and write that data to every connected client. For this we should track all sockets and handle connections and disconnections.

### dns module

node provides a dns native module. lookup can translate urls into IPs, it uses libuv the rest of the methods uses the network. reverse is the opposite of lookup.

### UDP sockets

In node we can use the native module dgram. With its own methods for creating and running the server.

```js
const dgram = require('dgram');
const server = dgram.createSocket('udp4');
server.bind(PORT, HOST);
```

The message event is called every time the client posts a message with the message and remote information.

```js
server.on('message', (msg, rinfo) => {
  console.log(`${rinfo.address}:${rinfo.port} - ${msg}`);
});
```

For creating clients we use the same module and initialization but run `.send`.

## Node for web server

Use http module. The request event is called with every request with the request and the response.

https is http over tls or ssl. node provides the https module to use it. It receives one object parameters with options such as key and cert

node can also be a http client using also the http module with functions like `http.request` and `http.get`

url.parse parses urls into the different segments or parts of a url. querystring module converts models into querystrings that can be used for urls. It also parses querystrings into objects.

## modules

os provides access to the os info. cpu, operating system, network interfaces, os version etc

fs filesystem operations, sync and async. Async are error first, sync throw errors. readdir reads a whole directory into an array of file names. truncate truncates file content. stat gives meta info about the file. utimes changes the timestamp. watch watches for events in a directory.

console. trace prints the call stack

util. deprecate can wrap a function that will show a deprecation warning.

node comes with a debugger that can be activated by `node debugger index.js`. sb adds breakpoints to line, cont continues, repl is used for inspecting, watch adds watchers to variables, list shows the current line that's being executed. --inspect will open the debugger in chrome.

## Streams

many built in modules implement streams. collections of data that might not be always available and dont have to fit in memory. It allows to avoid buffering big chunks of data in memory.

```js
const src = fs.createReadStream('./big.file');
src.pipe(res);
```

Readable, writable, duplex, transform

readable is the origin, writable is the destination, duplex are both and transform is a duplex that transforms the data. All streams are instances of event emitter. pipe can be chained (over duplexes).

readable can be paused or flowing (pull/push). In flowing mode we have to listen to events of that data. resume() and pause() methods switch between the states.

## Clusters and child processes

Multiprocess is the only scalability for node. For scaling an app we can clone the app, decompose its functionalities (microservice), split the app into instances (horizontal partitioning).

child_process module can run system commands in child processes. For creating child processes we can use spawn, fork, exec, execfile. They have sync versions.

```js
const { spawn } = require('child_process');

const child = spawn('find', ['.', '-type', 'f']);

child.stdout.on('data', (data) => {
  console.log(`child stdout:\n${data}`);
});

child.stderr.on('data', (data) => {
  console.error(`child stderr:\n${data}`);
});

child.on('exit', function (code, signal) {
  console.log(`child process exited with code ${code}, signal ${signal}`);
});
```

the spawn function returns a child process instance that implements the event emitter so we can register handlers for events like 'exit', disconnect, error, message (the child process uses process.send), close... Every child process has attached streams stdin, stdout, stderror, when those are close the process is closed and the close event is emitted. Those streams can have registered listeners to their events. The spawn function streams the data returned from the command.

child processes streams can be piped to share data among processes.

exec method creates a shell and buffers all the data returned from the command.

```js
const { exec } = require('child_process');

exec('find . -type f | wc -l', (err, stdout, stderr) => {
  if (err) {
    console.error(`exec error: ${err}`);
    return;
  }

  console.log(`Number of files ${stdout}`);
});
```

execfile executes a file and does not use a shell.

fork allows to exchange messages with the child process with process.send or child.send and listening to the message event in the process.

### cluster

enable load balancing and forks the application process as many time as cpu cores. Helps to implement the clonning strategy to one machine. the cluster module is native. A master process decides (round robin by default) which worker process takes the request. 1 master, multiple workers.

create a cluster file that if it's master will call fork to create as many processes as cpus in the machine and if it's a child will require the server file with the server configuration. When forking the cluster file it will be run in worker mode with the isMaster flag set to false.

```js
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) { // or !isWorker
  const cpus = os.cpus().length;

  console.log(`Forking for ${cpus} CPUs`);
  for (let i = 0; i<cpus; i++) {
    cluster.fork();
  }
} else {
  require('./server');
}
```

Each process has its own event loop, queue, call stack...

The master cluster can access its workers through cluster.workers (object) and send messages with worker.send(). In the workers (server.js) we can get the received messages by listening to process.on 'message'.

The master is notified whenever a child exits listening to the 'exit' envent. It can generate new workers when they crash. It can also kill or disconnect workers with those methods.

```js
  cluster.on('exit', (worker, code, signal) => {
    if (code !== 0 && !worker.exitedAfterDisconnect) {
      console.log(`Worker ${worker.id} crashed. ` +
                  'Starting a new worker...');
      cluster.fork();
    }
  });
```

The exitedAfterDisconnect indicates if the worker was disconnected by the master. We can use singals to let the master know that it has to disconnect the workers. In linux, signals sent to processes can be listened to as an event on process. process.on('SIGUR2'...

pm2 can be used to simplify cluster management.

State should be shared between workers (redis, db). You can also use sticky load balancing (if your load balance allows it), keep records on the master of which user talked with which worker, latter requests by that user will go to the same worker.
