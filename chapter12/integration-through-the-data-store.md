One of the most commonly used techniques to integrate different parts of an application has always been to share the same data store, along with the same code base. This is usually known as a monolithic application, often ending up with a single data-store that hosts the data related to all the concerns within the application.

Consider an e-commerce application, a shared data-store would contain all concerns \(eg: tables within a relational database\) surrounding the catalog, billing, inventory, and so on. There is nothing bad with this approach per se, for example in small linear applications were the complexity is not too high. However, within complex domains, some issues can arise. If you share data across many tables touching multiple application concerns, transactions will have a big impact on performance.

Another less technical problem that could develop will be in-regard to the Ubiquitous Language. The main advantage to the seperation of Bounded Contexts is having a single Ubiquitous Language for each one. In doing so, models will be separated into their own contexts. Mixing all models together within the same context can lead to ambiguity and confusion.

Going back to the e-commerce system, imagine we want to introduce the concept of a t-shirt. Within the catalogue context, a t-shirt would be a product with properties like color, size, material and maybe some fancy pictures. In the inventory system however, we do not really wish to concern

ourselves with these. A product here has a different meaning, were we care about different properties like weight, location in the warehouse or dimensions. Mixing both contexts together will tangle concepts and will complicate the design. In DDD terms, mixing concepts in this manner is what is called a Shared Kernel.

> Shared Kernel
>
> Designate some subset of the domain model that the teams agree to share. Of course this
>
> includes, along with this subset of the model, the subset of code or of the database design associated with that part of the model. This explicitly shared stuff has special status, and shouldn’t be changed without consultation with the other team. Integrate a functional system frequently, but somewhat less often than the pace of CONTINUOUS INTEGRATION within the teams. At these integrations, run the tests of both teams.
>
> Eric Evans - Domain-Driven Design, Tackling complexity in the heart of software. Chapter 14 - Shared Kernel

We do not recommend using a Shared Kernel. As multiple teams can collide within the development of it, ending up having maintenance issues and becoming a friction point. Changes in the Shared Kernel should be agreed upon beforehand, between all parties involved. Conceptually it has other problems, such as people seeing it as a bag to place ‘stuff’ that does not belong anywhere else, growing indefinitely.

A better way of dealing with the ever growing monolithic complexity is to break it up in different autonomous pieces. Such as communicating through REST, RPC or messaging systems. This requires drawing clear boundaries, with each context likely ending up with their own infrastructure – data stores, servers, messaging middleware, etc. – and even its own team. As you may foresee, this could lead to some degree of duplication. That is a trade-off that we are willing to make in order to reduce complexity. These autonomous pieces receive the name of Bounded Contexts.



集成应用程序不同部分的最常用技术之一始终是共享相同的数据存储和相同的代码库。这通常被称为单片应用程序，通常以单个数据存储结束，存储与应用程序中的所有关注点相关的数据。



考虑一个电子商务应用程序，共享数据存储将包含围绕目录、账单、库存等的所有关注事项\(例如:关系数据库中的表\)。这种方法本身没有什么不好，例如，在小型线性应用程序中，复杂性并不太高。然而，在复杂的领域中，可能会出现一些问题。如果跨多个表共享涉及多个应用程序问题的数据，事务将对性能产生很大影响。



另一个不太技术性的问题是关于无处不在的语言。分隔有界上下文的主要优点是为每种上下文使用一种普遍存在的语言。这样，模型将被分离到它们自己的上下文中。在同一上下文中将所有模型混合在一起会导致歧义和混淆。



回到电子商务系统，假设我们要引入t恤的概念。在产品目录中，t恤的属性包括颜色、尺寸、材质，可能还有一些花哨的图片。但是在库存系统中，我们并不是真的希望关注



自己与这些。这里的产品有不同的含义，我们关心的是不同的属性，比如重量，仓库的位置或者尺寸。将两种上下文混合在一起会使概念混乱，并使设计复杂化。在DDD术语中，以这种方式混合概念称为共享内核。



> 共享内核
>
>
>
> 指定团队同意共享的域模型的某个子集。当然,这
>
>
>
> 除了模型的这个子集之外，还包括与模型的那个部分相关联的代码子集或数据库设计子集。这种显式共享的东西具有特殊的状态，不应该在没有与其他团队协商的情况下更改。频繁地集成功能系统，但是比团队内持续集成的速度要慢一些。在这些集成中，运行两个团队的测试。
>
>
>
> 领域驱动的设计，解决软件核心的复杂性。第14章-共享内核

我们不建议使用共享内核。由于多个团队可能在it开发过程中发生冲突，最终导致维护问题并成为一个摩擦点。共享内核中的更改应该在所有相关方之间事先达成一致。从概念上讲，它还有其他问题，比如人们把它视为一个袋子，用来放置不属于其他任何地方的“东西”，并无限增长。



处理不断增长的整体复杂性的更好方法是将其分解为不同的自治部分。例如通过REST、RPC或消息传递系统进行通信。这需要明确界限，每个上下文都可能拥有自己的基础设施\(数据存储、服务器、消息传递中间件等\)，甚至拥有自己的团队。正如您可以预见的，这可能会导致某种程度的重复。为了减少复杂性，我们愿意做出这样的取舍。这些自治部分接受有界上下文的名称。

