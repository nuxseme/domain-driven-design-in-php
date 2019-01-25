Consider the differences in the Ubiquitous Language when we discuss the side effects from relocating a customer, the event makes the concept explicit where as previously the changes that would occur within an aggregate or between multiple aggregates were left as an implicit concept that needed to be explored and defined. As an example, in most systems the fact that a side effect occurred is simply found by a tool such as Hibernate or Entity Framework, if there is a change to the side effects of a use case, it is an implicit concept. The introduction of the event makes the concept explicit and part of the Ubiquitous Language; relocating a customer does not just change some stuff, relocating a customer produces a CustomerRelocatedEvent which is explicitly defined within the language.



考虑的差异无处不在的语言当我们讨论迁移客户的副作用,这一事件使概念明确,正如前面的变化会发生在一个聚合或多个总量之间都作为一个隐含的概念,需要探索和定义。例如，在大多数系统中，副作用的发生仅仅是由Hibernate或实体框架等工具发现的，如果用例的副作用发生了变化，那么它就是一个隐式概念。事件的引入使概念变得明确，成为普遍语言的一部分;重新定位客户不仅仅是改变一些东西，重新定位客户还会产生CustomerRelocatedEvent，它是在语言中显式定义的。

