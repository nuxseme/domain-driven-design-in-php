From https://en.wikipedia.org/wiki/Domain-driven\_design\#Building\_blocks\_of\_DDD²:

Aggregate: A collection of objects that are bound together by a root entity, otherwise known as an

aggregate root. The aggregate root guarantees the consistency of changes being made within the

aggregate by forbidding external objects from holding references to its members.

Example: When you drive a car, you do not have to worry about moving the wheels forward, making

the engine combust with spark and fuel, etc.; you are simply driving the car. In this context, the car

is an aggregate of several other objects and serves as the aggregate root to all of the other systems.

