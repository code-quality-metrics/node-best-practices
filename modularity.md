# Modular design 

> Modular programming is a software design technique that emphasizes separating the functionality of a program into independent, interchangeable modules

Modularity helps with:

- migration
- maintainability
- future development
- deployments
- reusability
- maximize productivity
- testing

We should prevent code bases from growing organically.

## Things to consider when designing a module

There are certain things we should take into account when deciding if a new feature should be a module or not.

### Comply with the [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).

The single responsibility principle is a computer programming principle that states that every module or class should have responsibility over a single part of the functionality provided by the software, and that responsibility should be entirely encapsulated by the class, module or function.

A module should be narrow and focused. It should have exclusive responsibility of a piece of the whole system. Every interaction with that piece should be handle by the module. 

### API design

A module should provide an interface for other modules or clients to interact with it. Pay special attencion to the design and definition of this interface (API). See [The Three Principles of Excellent API Design](https://nordicapis.com/the-three-principles-of-excellent-api-design/): Purpose, Usability and Constraints.

### Encapsulation

Limit the exposure of modules within their system. Only provide a controlled interface for access. The inner implementation should be changable withoutn interfiering with the clients. Control the dependencies between modules and minimize them to improve scalability. The network formed by the modules should be logical, easy to follow and simple.

### Avoid mono-dependencies

Submodules are a thing. If a module is only used by another module and won't be shared by any other modules maybe it should be part of its parent. Creating modules that have a 1-1 relation could generate mess in the system. Consider merging those modules or have one become a submodule.

### Be aware of context

Be aware of your system context. Sometimes too many modules don't make sense for small systems. Modularization should be subjective and should not be pursued if it won't increase clarity.

[5 essentials of modular design](https://medium.com/@shanebdavis/the-5-essential-elements-of-modular-software-design-6b333918e543)

For node: TODO:

- libs (external, separate directory)
- organize db models (other repo)
- feature modularization
