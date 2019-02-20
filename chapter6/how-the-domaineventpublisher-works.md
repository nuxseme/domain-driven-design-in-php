A DomainEventPublisher is a Singleton class available from our Bounded Context in order to publish DomainEvents. It also has support to attach listeners, DomainEventSubscriber, that will be listening for any DomainEvent they are interested in. This is not quite different than when subscribing with jQuery to an event using on method.

DomainEventPublisher是一个单例类，可以从我们的有界上下文中获得，以便发布DomainEvents。它还支持附加侦听器DomainEventSubscriber，该侦听器将侦听它们感兴趣的任何DomainEvent。这与使用on方法使用jQuery订阅事件没有太大区别。

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

方法publish遍历所有可能的订阅者，检查他们是否对发布的域事件感兴趣。如果是这种情况，则调用订阅者的方法句柄。

订阅方法添加了一个新的DomainEventSubscriber，它将侦听特定的域事件类型。

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

正如我们已经讨论过的，持久化所有域事件是一个好主意。如何轻松持久化app中发布的所有domainevent ?为此使用特定订阅服务器。让我们创建一个DomainEventSubscriber，它将侦听所有DomainEvents，无论它是什么类型的，并使用我们的EventStore持久化它们。

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

$eventStore可以是一个定制的规则存储库\(如前所述\)，也可以是任何其他能够将DomainEvent持久化到数据库中的对象。

