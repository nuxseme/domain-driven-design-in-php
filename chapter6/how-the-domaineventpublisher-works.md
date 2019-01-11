A DomainEventPublisher is a Singleton class available from our Bounded Context in order to publish DomainEvents. It also has support to attach listeners, DomainEventSubscriber, that will be listening for any DomainEvent they are interested in. This is not quite different than when subscribing with jQuery to an event using on method.

```php
class DomainEventPublisher
{
    private $subscribers;
    private static $instance = null;

    public static function instance()
    {
        if (null === static::$instance) {
            static::$instance = new static();
        }

        return static::$instance;
    }

    private function construct()
    {
        $this->subscribers = [];
    }

    public function clone()
    {
        throw new \BadMethodCallException('Clone is not supported');
    }

    public function subscribe(DomainEventSubscriber $aDomainEventSubscriber)
    {
        $this->subscribers[] = $aDomainEventSubscriber;
    }

    public function publish(DomainEvent $anEvent)
    {
        foreach ($this->subscribers as $aSubscriber) {
            if ($aSubscriber->isSubscribedTo($anEvent)) {
                $aSubscriber->handle($anEvent);
            }
        }
    }
}
```

The method publish goes through all the possible subscribers, checking if they are interested in the published Domain Event. If that’s the case, the method handle of the subscriber is called.

The method subscribe adds a new DomainEventSubscriber that will be listening to specific Domain Event types.

```php
interface DomainEventSubscriber
{
    /**
     * @param DomainEvent $aDomainEvent
     */
    public function handle($aDomainEvent);

    /**
    @param DomainEvent $aDomainEvent
    @return bool
     */
    public function isSubscribedTo($aDomainEvent);
}
```

As we have already discussed, persisting all the Domain Events is a great idea. How can we easily persist all the DomainEvents published in our app? Using an specific subscriber for that. Let’s create a DomainEventSubscriber that will listen to all DomainEvents, no matter what type, and persists them using our EventStore.





```php

class PersistDomainEventSubscriber implements DomainEventSubscriber
{
    private $eventStore;

    public function construct(EventStore $anEventStore)
    {
        $this->eventStore = $anEventStore;
    }

    public function handle($aDomainEvent)
    {
        $this->eventStore->append($aDomainEvent);
    }

    public  function  isSubscribedTo($aDomainEvent)
    {
        return true;
    }
}

```

$eventStore could be a custom Doctrine repository, as already seen, or any other object capable of persisting DomainEvent into a Database.



