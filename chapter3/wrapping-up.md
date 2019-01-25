Using Value Objects for modeling concepts in your Domain that measure, quantify or describe is highly recommended. As shown, Value Objects are easy to create, maintain and test. In order to handle persistence within a DDD application, using an ORM is a must. However, in order to persist Value Objects using Doctrine without current support for embedded values \(scheduled for version 2.5\), there are two options: adding the Value Object fields directly into your Entity and mapping them \(less elegant, but easier\) or extending your entities \(far more elegant, but more complex\).

强烈推荐使用值对象对领域中度量、量化或描述的概念进行建模。如图所示，值对象易于创建、维护和测试。为了在DDD应用程序中处理持久性，必须使用ORM。然而，为了在不支持嵌入式值的情况下使用Doctrine持久化值对象\(计划在2.5版本中实现\)，有两个选项:直接将值对象字段添加到实体中并映射它们\(不太优雅，但更容易\)，或者扩展实体\(优雅得多，但更复杂\)。

