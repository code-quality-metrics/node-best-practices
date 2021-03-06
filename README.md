# Node and Express Best practices

Exploring and explaining Node JS and Express best practices and architecture designs.

Other resources:

- [Overview of the basics about JS and Node](js_and_node.md)
- [Node as a web server](web_server.md)
- [Modularity](modularity.md)
- [Testing](testing.md)

## Node principles

- Event driven
  Programming paradigm in which the program execution is determined by events. Each function executes an event. Node provides the event listener that awaits for calls.
- Non-blocking I/O
  When Node.js needs to perform an I/O operation instead of blocking the thread and wasting CPU cycles waiting, Node.js will resume the operations when the response comes back. This allows Node.js to handle thousands of concurrent connections.
- Single process
  Node.js app is run in a single process, without creating a new thread for every request. Node sends requests in an event loop and goes on to handle the next request in the call stack.
- Small core
  Node has a simple and small core leaving the bulk of features to npm packages.
- High [modularity](modularity.md)
  The Node.js way, in fact, involves extreme levels of reusability. Applications are composed of a high number of small, well-focused dependencies.

<https://nodejs.dev/introduction-to-nodejs>

<https://simpleprogrammer.com/top-4-javascript-concepts-a-node-js-beginner-must-know/>

## The event loop

One instance, one thread. The event loop doesn’t get generated instantly as soon as we run our program. In fact, it only runs once the whole program has been executed. Node looks at its inner collection of pending OS tasks and selects the next one on each tick. The event loop is composed of a series of phases, each with its own specific tasks, all processed in a circular repetitive way

Node provides multithreading through libraries like fs module. They run outside the node thread.

<https://blog.logrocket.com/a-complete-guide-to-the-node-js-event-loop/>

<https://www.youtube.com/watch?v=PNa9OMajw9w>

How to break it

- long tasks.
  Node.js server allows one operation to monopolize all resources in the process. If it is too long other tasks won`t take any thread space. This happens with long sync methods. Don't use fs.readFileSy.
  Long tasks could also happen due to unlimitting or not validating inputs.
  Try to keep high traffic endpoints from being too heavy.
  <https://blog.scottnonnenberg.com/breaking-the-node-js-event-loop/>

## Best practices

- Componetize the code base, create npm packages for common utilities (private repo). Keep a lib directory within your codebase with utils.
- Layer up the app (middlewares are embraced), leave Express just for the api layer.
- Protect env vars, throw errors if not present. Keep secrets out of the repo. process.env should only be accessed through a centralized config file.
- Use async await (forget callbacks). Don't leave unhandled promises.
- Keep errors as native Errors, don't go with customs ones.
- Document your api, models and errors. (Graphql) or autogenerated
- Dont use console logs, use a dedicated logger library. Monitor errors.
- Always validate inputs (Joi).
- ESLint, pretify, naming conventions. Use lowerCamelCase when naming constants, variables and functions and UpperCamelCase
- All requires at the top of the files.
- Dont reinvent the wheel, use npm
- Decouple api from frontend.
- Use semantic versioning for your modules: Major.Minor.Patch 1.9.12 <https://semver.org>

<https://medium.com/swlh/common-mistakes-that-node-js-developers-make-9df7106d09f1>

<https://github.com/goldbergyoni/nodebestpractices>

## Testing:

- Unit testing, coverage
- Integration / acceptance. This one is the most important, keeps consistency and reliablity.
- Penetration, performance, dependencies,

More about testing: [Testing](testing.md)

## Security

- Implement rate limiting using an external service
- Follow standards for session management (keep it in a middleware) oauth etc. Provide easy revocabilty.
- Use bcrypt for passwords (salt)
- Validate JSONs
- Provide visibility of all entrypoints to the service (login, auth...) limit calls agains brute force.
- Take care of errors not giving up PI (this email is not in our system). Dont return whole errors to the client.
- Careful with regex validation. Evil regex could take seconds to validate and block the event loop.
- Hide your tech stack <https://en.wikipedia.org/wiki/Denial-of-service_attack>
- Take care of secrets and .env files.
- Prevent introducing user input into `eval()`, `setTimeout()` and `setInterval()` functions. This could cause the execution of said input.
  <https://medium.com/@nodepractices/were-under-attack-23-node-js-security-best-practices-e33c146cb87d>
  <https://www.npmjs.com/package/helmet>

Express recommendations and useful packages: <https://expressjs.com/en/advanced/best-practice-security.html>

TODO:

- file structure
- express
- past experiences
- auth
- db and models
- feature modules or not...
