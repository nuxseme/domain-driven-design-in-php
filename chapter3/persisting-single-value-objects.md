Many different options are available to persist a single Value Object. These range from using Serialize LOB or Embedded Value as mapping strategies, to use an ad-hoc ORM or an open-source alternative, such as Doctrine. We consider an ad-hoc ORM to be a custom built ORM that your company may have developed in order to persist Entities in a database. In our scenario, the ad-hoc ORM code is going to be implemented using the DBAL¹¹ library. The Doctrine database abstraction and access layer \(DBAL\) offers a lightweight runtime around a PDO-like API, along with additional features such as, database schema introspection and manipulation through an OO API.



可以使用许多不同的选项来持久化单个值对象。这些策略的范围从使用序列化LOB或嵌入式值作为映射策略，到使用专用ORM或开源替代方案\(如Doctrine\)。我们认为一个特别的ORM是一个定制构建的ORM，您的公司可能已经开发了这个ORM来将实体持久化到数据库中。在我们的场景中,特别的ORM代码是实现使用DBAL¹¹图书馆。Doctrine数据库抽象和访问层\(DBAL\)围绕类似于pdo的API提供了轻量级运行时，还提供了其他特性，如通过OO API进行数据库模式内省和操作

