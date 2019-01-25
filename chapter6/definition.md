Domain Events are one specific type of event used for notifying Domain changes to local or remote Bounded Contexts.

Vaughn Vernon defines² a Domain Event as as

> an occurrence capture of something what happened in the domain.

Eric Evans defines³ a Domain Event as

> a full-fledged part of the domain model, a representation of something that happened in the domain. Ignore irrelevant domain activity while making explicit the events that the domain experts want to track or be notified of, or which are associated with state change in the other model objects.

Martin Fowler defines⁴ a Domain Event as 

> captures the memory of something interesting which affects the domain.

Examples of Domain Events in a Web application are UserRegistered, OrderPlaced, UserRelocated or ProductAdded.



领域事件是一种特定类型的事件，用于将领域更改通知到本地或远程有界上下文。

弗农·沃恩²域事件定义为

> 领域中发生可捕获的事件

Eric Evans³域事件定义为

> 领域模型的成熟部分，表示领域中发生的事情。忽略不相关的领域活动，同时显式地显示领域专家希望跟踪或通知的事件，或者与其他模型对象中的状态更改相关的事件。

Martin Fowler⁴域事件定义为

> 可捕获的对领域有影响的感兴趣的东西

Web应用程序中的域事件的例子有userregistration、orderplace和userlocate或ProductAdded。



