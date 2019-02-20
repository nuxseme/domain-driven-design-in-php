The mapping between your domain objects and the database can be considered an implementation detail. The domain life-cycle should not be aware of these persistence details. As such, the mapping information should be defined as part of the infrastructure layer, outside the domain and as the implementation for the repositories.

域对象和数据库之间的映射可以看作是实现细节。域生命周期不应该知道这些持久性细节。因此，映射信息应该定义为基础结构层的一部分，在域之外，并作为存储库的实现。

