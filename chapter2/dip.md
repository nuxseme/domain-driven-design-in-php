How can we fix this? As the Domain Model layer depends on concrete infrastructure implementations,

the Dependency Inversion Principle⁷ could be applied by relocating the Infrastructure layer on

top of the other three layers.

如何处理？领域模型层依赖基础设施的层的实现。依赖倒置原则可以是基础设施移到其他3层的顶部

> The Dependency Inversion Principle
>
> High level modules should not depend upon low level modules. Both should depend upon
>
> abstractions.
>
> Abstractions should not depend upon details. Details should depend upon abstractions.
>
> Robert C. Martin
>
> 依赖倒置原则
>
> 高级模块\(业务复杂,调用其他模型层 \)不应该依赖低级模块，两者都应该依赖抽象。抽象不应该依赖细节，细节应该依赖抽象

By using the Dependency Inversion Principle, the architecture schema changes and the Infrastructure

layer which can be referred to as low level modules now depend on the UI, the Application Layer

and the Domain Layer, which are the high level modules. The dependency has been inverted.

But then, what is Hexagonal Architecture?, and how does it fit within of all this? Hexagonal

Architecture \(also known as Ports and Adapters\) was defined by Alistair Cockburn and represents

the application as an hexagon where each side represents a Port with one or more Adapters. A

Port is a connector with a pluggable Adapter which transforms an outside input to something the

inside application can understand. In terms of the DIP, the Port would be a high level module and an

Adapter would be a low level module. Furthermore, if the application needs to emit some message to

the outside it will also use a Port with an Adapter to send it and transform it to something that the

outside can understand. For this reason, Hexagonal Architecture brings up the concept of symmetry

in the application and it is also the main reason why the schema of the architecture changes. It is

often represented as a hexagon, because it does no longer make sense to talk about a “top” layer

nor a “bottom” layer. Instead, Hexagonal Architecture talks mainly in terms of the ‘outside’ and the

‘inside’.

通过使用依赖倒置，架构模式变更了。低层模块依赖UI,应用层和Domain层成为了高层，依赖被倒置了。那什么是六边形架构呢？它是如何适配的？六边形架构（端口和适配器\)由Alistair Cockburn定义。一个端口是带有可插拔适配器的连接器，该适配器将外部输入转换为内部应用程序可以理解。在DIP方面，该端口将是一个高级模块和一个适配器将是一个低级模块。它还将使用带有适配器的端口来转换并发外部可以理解消息。因此，六边形建筑提出了对称的概念在应用中，它也是架构模式发生变化的主要原因。它是通常用六边形表示，因为讨论“顶层”已经没有意义了也不是“底层”。相反，六边形的建筑主要是用“外部”和“内部”来区分。

---

抛开概念，DIP其实就是用来减少依赖细节的。转而依赖抽象，依赖接口。在由子类继承实现接口。而且抽象的类表示一个整体的概念，适合在代码中传递。

