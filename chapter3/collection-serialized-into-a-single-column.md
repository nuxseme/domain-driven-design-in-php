Serializing a collection of Value Objects into a single column is most likely the easiest solution. Everything that has previously been discussed through persisting a single Value Object applies in this situation. With Doctrine you can use an Object or Custom Type, with some additional considerations to bear in mind: Value Objects should be small in size, however, if you wish to persist a large collection, be sure to consider the maximum column length and the max row width that your database engine can handle.

将一组值对象序列化到单个列中可能是最简单的解决方案。前面通过持久化单个值对象讨论的所有内容都适用于这种情况。与教义或自定义类型,您可以使用一个对象与其他一些注意事项要记住:值对象应该是小,但是,如果你想坚持一个大集合,一定要考虑的最大列长度和最大行宽度可以处理您的数据库引擎。

> Exercise
>
> Think up both Doctrine Object Type and Doctrine Custom Type implementation strategies for persisting a Product with different prices.
>
> 为不同价格的产品制定原则对象类型和原则自定义类型实现策略。



