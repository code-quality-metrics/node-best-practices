# NodeJS as a web server

Information sent from the server is defined by protocols like HTTP, FTP, SMTP... and the act of separating that information into Packets and sending it through a socket is the TCP/IP protocol.

HTTP Parser is a c program embedded in node. It is wrapped in the js side of the code and several features are added to it. Node can build the HTTP format for requests and responses using the http-parser (among other modules).

Using the http native module node allow us to build web servers. The http.createServer has a function parameter called when a request happens with the request and the response. Request and Response also inherit from EventEmitter.

```js
const http = require('http')
const server = http.createServer(function(req, res) {})
server.listen(3000)
```

Responses in node can be used as a writeable stream and data can be piped to them (e.g. reading an html file and writing it to the response as a stream)

## Express

Minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications.

Calling express returns a function with properties and methods added to it, it gets wraps of the response and requests as paramenters. The express app provides a listen function that wraps the http `server.listen`.

```js
const expres = require('express')
const app = express()
app.listen(3000)
```

### Routing

Express Routing allows patterns and regular expressions in the urls.

```js
app.get('/ab?cd', function (req, res) {
  res.send('ab?cd')
})
app.get(/.*fly$/, function (req, res) {
  res.send('/.*fly$/')
})
```

Url parameters are passed to `request.params` the name is defined in the route with ":" as `/user/:id`
Query strings are available at `request.query`.
For accessing bodies, use body-parser and it will be availabe at `request.body`.

### Middleware

Middlewares work through `app.use` it takes an url and a function with request, response and next or just the function.

express.static streams static files automatically. All files under public directory will be returned when requested throught the assets endpoint.

```js
app.use('/assets', express.static(_dirname + '/public'))
```

<https://expressjs.com/en/resources/middleware.html>

### Template engines

Many different engines can be used with express (that's why it's unopinionated). The selected engine is setted in the app.js. Express looks for templates in /views by default.

```js
app.set('view engine', 'ejs')
```

Rendering templates can be done by calling `response.render` express will handle it and apply the rendering depending on the selected engine.

```js
res.render('index', {name: user.name})
```

### Async await in routes

TODO:

## REST

Representational State Transfer (REST) software architecture style for web services. Independent of the payload format. When we use REST style an Application can interact with resource by knowing only two things :

- Identifier of the resource
- Action to be performed on the resource

GET person/id
POST person

## DBs

TODO:

MySQL, Mongo (with mongoose)
Mongoose provides schemas on top of mongo.
Check mongo lab
