  
In order to be able to build complex applications, one of the key requirements is having an

architectural design that fits the applications needs. A good advantage of Domain-Driven Design is

that it is not tied to any particular architecture style. Instead, we are free to choose the architecture

that best fits the needs of every Bounded Context inside the Core Domain, offering a diverse set of

architectural choices for every specific domain problem.

For example, an Order Processing System can use Event Sourcing to track all the different order

operations, a Product Catalog can use CQRS to expose the product details to the different clients

and a Content Management System can use plain Hexagonal Architecture to expose requirements

such as blogs, static pages, and so on.

This chapter presents an introduction to every relevant architecture style in the land of PHP, through

an evolution from traditional ‘old-school’ PHP code to a more sophisticated architecture. Please note

that although there are many other existing architecture styles, like Data Fabric or SOA, we found

them a bit complex to introduce from the PHP perspective.



---

构建复杂应用的关键是选个一个适合的架构设计。DDD不依赖任何架构风格，相反我们估计在不同的边界上下文中采用不同的合适的架构来解决不同的问题。架构无关性是DDD的一大特色。举个例子，订单是系统可采用事件溯源去追踪订单操作记录，产品目录可以使用CQRS为不同的客户端提供产品详情，内容管理系统可以使用六边形架构去适配不同的终端

本章主要讲述PHP领域下的各种架构风格，从老派的架构到更复杂的架构的演进。虽然存在其他优化的架构但是从php的角度不好解释，比如SOA ,Data Fabric

