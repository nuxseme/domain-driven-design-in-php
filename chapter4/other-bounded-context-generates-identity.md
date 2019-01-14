Probably this would be the most complex identity generation strategy, because it enforces to have a local entity to be dependent not only on local bounded context events, but in external bounded contexts events. So in terms of maintenance, the cost would be high.

Other Bounded Context provides of some UI widget to select the identity of the local entity. This can even grab some properties of the remote entity to its own.

When synchronization is needed between the entities of the Bounded Contexts, usually can be achieved with an Event Driven architecture on each of the Bounded Context that need to be notified when the original entity is changed.



