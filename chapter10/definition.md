Martin Fowler defines¹ a Repository as

the mechanism between the domain and data mapping layers, acting like an in-memory domain object collection. Client objects construct query specifications declaratively and submit them to Repository for satisfaction. Objects can be added to and removed from the Repository, as they can from a simple collection of objects, and the mapping code encapsulated by the Repository will carry out the appropriate operations behind the scenes. Conceptually, a Repository encapsulates the set of objects persisted in a data store and the operations performed over them, providing a more object-oriented view of the persistence layer. Repository also supports the objective of achieving a clean separation and one-way dependency between the domain and data mapping layers.



Martin Fowler将¹存储库定义为

域和数据映射层之间的机制，类似于内存中的域对象集合。客户端对象以声明的方式构造查询规范，并将其提交到存储库以获得满足。对象可以添加到存储库中，也可以从存储库中删除，就像它们可以从简单的对象集合中删除一样，存储库封装的映射代码将在幕后执行适当的操作。从概念上讲，存储库封装了持久存储在数据存储中的对象集和在其上执行的操作，提供了持久层更面向对象的视图。Repository还支持在域和数据映射层之间实现完全分离和单向依赖的目标。

