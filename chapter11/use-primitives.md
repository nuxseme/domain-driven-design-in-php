We recommend using just basic types to build up your request objects. That means string, integers, booleans, and so on. We are just abstracting away input parameters. You should be able to consume Application Services independently from the delivery mechanism. Even pretty complicated HTML forms get translated into basic types all the time at the controller level. You don’t want to mangle your framework and your business logic together.

On some scenarios is tempting to use Value Objects directly. Don’t do it, updates on the Value Object definition will affect all clients. You’ll be coupling clients with your Domain logic.



