A Repository is a mechanism that acts as a storage location. The difference between a DAO and a Repository is that a DAO follows a database-first approach, decreasing cohesion with many low- level methods to query the database. Depending on the underlying persistence mechanics, we’ve seen different Repository approaches:



* Collection-Oriented repositories tend to be more pure to the domain model, even if they persist entities. From the client’s point of view, it looks like a collection \(Set\). There’s no need for explicit persistence calls on Entity updates, as the repository tracks changes on the objects. We explored Doctrine as the underlying persistence mechanism for this type of repository as it provides automatic changes monitoring on objects \(Unit of Work\).
* Persistence-Oriented repositories require explicit persistence calls as they don’t track object changes. We explored Redis and plain SQL implementations.



Along the way, we discovered Specifications as a pattern that help us querying the database without sacrificing flexibility and cohesion. We also studied how to manage Transactions and how to test our services with simple and fast in-memory Repository implementations.



