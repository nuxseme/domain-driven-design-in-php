Where do we persist Domain Events? In an Event Store. An Event Store is a DomainEvent repository that lives in our Domain space as an abstraction \(interface or abstract class\) its responsibility is to append Domain Events and query them. A possible basic interface could be:

我们在哪里持久化域事件?在事件存储中。事件存储是作为抽象\(接口或抽象类\)存在于域空间中的DomainEvent存储库，它的职责是追加域事件并查询它们。一个可能的基本接口是:

```php
interface EventStore
{
    public function append(DomainEvent $aDomainEvent);
    public function allStoredEventsSince($anEventId);
}
```

However, depending on the usage of your DomainEvents, the previous interface can have more methods to query your events.

In terms of implementation, you can decide to use a Doctrine repository, a DBAL one or a plain PDO. Because DomainEvents are immutable using a Doctrine one adds an unnecessary performance penalty. For a small to medium application, probably Doctrine is ok. Let’s see a possible implementation with Doctrine.

然而，根据DomainEvents的使用情况，前面的接口可以有更多的方法来查询事件。

在实现方面，您可以决定使用Doctrine存储库、DBAL存储库或普通PDO。由于DomainEvents使用原则是不可变的，因此增加了不必要的性能损失。对于一个中小企业的应用，或许学说是可以的。让我们看看原则的可能实现。

```php
class DoctrineEventStore extends EntityRepository implements EventStore
{
    private $serializer;

    public function append(DomainEvent $aDomainEvent)
    {
        $storedEvent = new StoredEvent( get_class($aDomainEvent),
            $aDomainEvent->occurredOn(),
            $this->serializer()->serialize($aDomainEvent, 'json')
        );

        $this->getEntityManager()->persist($storedEvent);
    }

    public function allStoredEventsSince($anEventId)
    {
        $query = $this->createQueryBuilder('e');
        if ($anEventId) {
            $query->where('e.eventId > :eventId');
            $query->setParameters(array('eventId' => $anEventId));
        }
        $query->orderBy('e.eventId');

        return $query->getQuery()->getResult();
    }

    private function serializer()
    {
        if (null === $this->serializer) {
            /** \JMS\Serializer\Serializer\SerializerBuilder */
            $this->serializer = SerializerBuilder::create()->build();
        }

        return $this->serializer;
    }
}
```

StoredEvent is the Doctrine Entity needed to map with the database. As you may have seen, when appending and after persisting the Store, there is no flush call. If this operation is inside a Doctrine transaction, this is not needed. So, let’s leave it without the call and we’ll go into more details when talking about Application Services. Let’s see the StoredEvent implementation.

StoredEvent是与数据库映射所需的Doctrine实体。正如您可能已经看到的，在追加存储和持久化存储之后，不存在刷新调用。如果该操作在Doctrine事务中，则不需要此操作。所以，让我们不要调用它，在讨论应用程序服务时，我们将更详细地讨论它。让我们看看StoredEvent实现。

```php
class StoredEvent implements DomainEvent
{
    private $eventId; private $eventBody; private $occurredOn; private  $typeName;

    /**
    @param string $aTypeName
    @param \DateTime $anOccurredOn
    @param string $anEventBody
     */
    public  function construct($aTypeName,  \DateTime  $anOccurredOn,  $anEventBody)
    {
        $this->eventBody = $anEventBody;
        $this->typeName  =  $aTypeName;
        $this->occurredOn = $anOccurredOn;
    }

    public function eventBody()
    {

        return $this->eventBody;
    }

    public function eventId()
    {
        return $this->eventId;
    }

    public  function typeName()
    {
        return  $this->typeName;
    }

    public function occurredOn()
    {
        return $this->occurredOn;
    }
}
```

And its mapping.

```php
Ddd\Domain\Event\StoredEvent: type: entity
table: event
repositoryClass: Ddd\Infrastructure\Application\Notification\DoctrineEventStore id:
eventId:
type: integer column: event_id generator:
strategy: AUTO fields:
eventBody:
column: event_body type: text
typeName:
column: type_name type: string length: 255
occurredOn:
column: occurred_on type: datetime
```

Because every DomainEvent may have different fields, we need to persist them serialized. typeName identifies the DomainEvent domain-wide. An Entity or Value Object has sense inside a BC but DomainEvents define a communication protocol between BC.

In distributed systems, s\*\*\* happens. You will have to deal with DomainEvents that are not published, lost somewhere in the chain or DomainEvents that are published more than once. That’s why it’s important to persist a DomainEvent with an id, so it’s easy to track which DomainEvents have been published and which are the missing ones.

因为每个DomainEvent可能有不同的字段，所以我们需要将它们序列化。typeName标识域范围内的DomainEvent。实体或值对象在BC中具有意义，但DomainEvents定义BC之间的通信协议。

在分布式系统中，s\*\*\*会发生。您将不得不处理未发布的domainevent、在链中的某个位置丢失的domainevent或已发布多次的domainevent。这就是为什么使用id持久化DomainEvent很重要，这样就很容易跟踪哪些DomainEvent已经发布，哪些是缺失的。

