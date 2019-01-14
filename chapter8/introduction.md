Aggregates are probably the most difficult to understand and implement building blocks of Domain-

Driven Design. For properly implement them, we need to understand concepts such as transaction,

locking and concurrency strategies. It’s also interesting to understand their origin and how the

NoSQL movement has influenced them so much.

From Vaughn Vernon’s “Implementing Domain-Driven Design”: Aggregates are carefully crafted

consistency boundaries that cluster Entities and Value Objects. Another amazing book, you should

definitely buy and read, is “NoSQL Distilled: A Brief Guide to the Emerging World of Polyglot

Persistence” by Pramod J. Sadalage. From this book: “In Domain-Driven Design, an aggregate is

a collection of related objects that we wish to treat as a unit. In particular, it is a unit for data

manipulation and management of consistency. Typically, we like to update aggregates with atomic

operations and communicate with our data storage in terms of aggregates.”

