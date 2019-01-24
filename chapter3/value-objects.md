Value Objects are a fundamental building block in Domain-Driven Design, used to model concepts of your Ubiquitous Language in code. A Value Object is not just a thing in your domain, it measures, quantifies, or describes something. They can be seen as small, simple objects such as money or a date range - whose equality is not based on identity, but instead on the content held.

For example, a product price could be modelled using a Value Object. In this case it is not representing a thing, but instead a value that allows us to measure how much money a product is worth. The memory footprint for these objects is trivial to determine \(calculated by their constituent parts\) and very little overhead. As a result, new instance creation is favoured over reference reuse, even when being used to represent the same value. Equality is then checked based on the comparability of both instances fields.

值对象是领域驱动设计中的基本构建块，用于为代码中无处不在的语言的概念建模。值对象不仅仅是域中的一个事物，它还度量、量化或描述某些事物。它们可以被看作是小的、简单的对象，比如钱或日期范围——它们的相等不是基于身份，而是基于所持有的内容。

例如，可以使用值对象建模产品价格。在这种情况下，它不代表一件东西，而是一个价值，让我们能够衡量一个产品的价值。这些对象的内存占用非常小\(通过它们的组成部分计算\)，开销也非常小。因此，新实例的创建比引用重用更受欢迎，即使是在用于表示相同的值时也是如此。然后根据两个实例字段的可比性检查相等性。

---

1. 不变性
2. 相等性
3. 描述性
4. 不可重用
5. 如何将一个概念划归为实体或者值对象？ （上下文）



