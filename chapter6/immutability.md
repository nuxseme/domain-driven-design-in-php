Domain Events describe changes in your Domain that have already happened. They talk about the past. By definition, it’s impossible to change the past, except if you are Marty McFly and have a Delorean, but that might be not the case. Domain Events are immutable, that’s it.



> Symfony Event Dispatcher
>
>     Some PHP frameworks support events. However, don’t confuse those events with Domain Events. They are different in characteristics and goals. For example, Symfony has the Event Dispatcher component. If you need to implement an event system for a state machine, for example, you can rely on it. The whole request to response Symfony trip is based in events too. However, Symfony Events are mutable, each of the listeners are capable of modifying the event to add or update the information in it.



