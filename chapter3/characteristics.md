Whilst modelling an Ubiquitous Languages concept in code, you should always favour Value Objects over Entities. Value Objects are easier to create, test, use and maintain.

With this in mind, you can decide on whether the concept in question could be modelled as a Value Object if…

It measures, quantifies, or describes a thing in the domain

It can be kept immutable

It models a conceptual whole, by composing related attributes as an integral unit

It is completely replaceable when the measurement or description changes

It can be compared with others through value equality

It supplies its collaborators with Side-Effect-Free behaviour

在为代码中普遍存在的语言概念建模时，应该始终优先考虑值对象而不是实体。值对象更易于创建、测试、使用和维护。

考虑到这一点，您可以决定是否可以将所讨论的概念建模为一个值对象，如果……

* 它度量、量化或描述域中的事物
* 它可以保持不变
* 它通过将相关属性组合为一个完整的单元来为概念整体建模
* 当测量或描述发生变化时，它是完全可替换的
* 它可以通过价值平等与他人进行比较
* 它为其合作者提供无副作用的行为



