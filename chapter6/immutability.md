Domain Events describe changes in your Domain that have already happened. They talk about the past. By definition, it’s impossible to change the past, except if you are Marty McFly and have a Delorean, but that might be not the case. Domain Events are immutable, that’s it.

域事件描述域中已经发生的更改。他们谈论过去。按照定义，改变过去是不可能的

> Symfony Event Dispatcher
>
> Some PHP frameworks support events. However, don’t confuse those events with Domain Events. They are different in characteristics and goals. For example, Symfony has the Event Dispatcher component. If you need to implement an event system for a state machine, for example, you can rely on it. The whole request to response Symfony trip is based in events too. However, Symfony Events are mutable, each of the listeners are capable of modifying the event to add or update the information in it.
>
> 一些PHP框架支持事件。但是，不要将这些事件与领域事件混淆。他们有不同的特点和目标。例如，Symfony具有事件分派器组件。例如，如果您需要为状态机实现事件系统，您可以依赖它。响应Symfony trip的整个请求也是基于事件的。然而，Symfony事件是可变的，每个侦听器都能够修改事件来添加或更新其中的信息。



