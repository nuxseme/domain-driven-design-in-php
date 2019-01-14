Where do we persist Domain Events? In an Event Store. An Event Store is a DomainEvent repository that lives in our Domain space as an abstraction \(interface or abstract class\) its responsibility is to append Domain Events and query them. A possible basic interface could be:

```php
interface EventStore
{
    public function append(DomainEvent $aDomainEvent);
    public function allStoredEventsSince($anEventId);
}
```

However, depending on the usage of your DomainEvents, the previous interface can have more methods to query your events.

In terms of implementation, you can decide to use a Doctrine repository, a DBAL one or a plain PDO. Because DomainEvents are immutable using a Doctrine one adds an unnecessary performance penalty. For a small to medium application, probably Doctrine is ok. Let’s see a possible implementation with Doctrine.

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



