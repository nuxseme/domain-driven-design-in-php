A Repository is a mechanism that acts as a storage location. The difference between a DAO and a Repository is that a DAO follows a database-first approach, decreasing cohesion with many low- level methods to query the database. Depending on the underlying persistence mechanics, we’ve seen different Repository approaches:

* Collection-Oriented repositories tend to be more pure to the domain model, even if they persist entities. From the client’s point of view, it looks like a collection \(Set\). There’s no need for explicit persistence calls on Entity updates, as the repository tracks changes on the objects. We explored Doctrine as the underlying persistence mechanism for this type of repository as it provides automatic changes monitoring on objects \(Unit of Work\).
* Persistence-Oriented repositories require explicit persistence calls as they don’t track object changes. We explored Redis and plain SQL implementations.

Along the way, we discovered Specifications as a pattern that help us querying the database without sacrificing flexibility and cohesion. We also studied how to manage Transactions and how to test our services with simple and fast in-memory Repository implementations.



存储库是一种充当存储位置的机制。DAO和存储库之间的不同之处在于，DAO遵循数据库优先的方法，降低了与许多查询数据库的低层方法的内聚性。根据底层持久性机制的不同，我们看到了不同的存储库方法:

* 面向集合的存储库往往对域模型更纯，即使它们保存实体也是如此。从客户机的角度来看，它看起来像一个集合\(集合\)。不需要对实体更新进行显式持久性调用，因为存储库跟踪对象上的更改。我们探讨了作为这种类型存储库的底层持久性机制的Doctrine，因为它提供了对对象\(工作单元\)的自动更改监视。
* 面向持久性的存储库需要显式持久性调用，因为它们不跟踪对象更改。我们研究了Redis和普通SQL实现。

在此过程中，我们发现规范作为一种模式帮助我们在不牺牲灵活性和内聚性的情况下查询数据库。我们还研究了如何管理事务以及如何使用简单而快速的内存存储库实现测试服务。

