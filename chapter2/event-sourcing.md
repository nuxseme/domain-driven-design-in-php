CQRS is a powerful and flexible architecture. There is an added benefit in regard to gathering and saving the d    omain events \(which occurred during an aggregate operation\), giving you a high-level degree of detail of what i    s going on within your domain. Domain Events are one of the key tactical patterns because of their significance     within the domain, as they describe past occurrences. An ever growing number of events is a smell of the busin    ess overlooking insight in the Domain. By using CQRS we gained a highly sophisticated history of all the releva    nt occurrences at a level that the whole state of the domain model can be expressed just by reproducing domain     events. We just need a tool for storing all those events in a consistent way. This store is called an eventstor  eventstore.

The fundamental idea behind Event Sourcing is to express the state of Aggregates as a linear sequence of events.

With CQRS we partially achieved the following - the Post entity alters its state by using domain events but it is persisted as always, mapping the object to a database table. Event Sourcing takes this a step further. If we were using a database table to store the state of all the blog posts and another to store the state of all the blog post comments and so on, by using Event Sourcing we could use just a single database table. A single append-only database table that would store all the domain events published by all the aggregates within the domain model. Yes, you have read that correctly, a single database table. With this model in mind, tools like object-relational mappers are not needed any more. They would be an overkill for a single database table. The only tool needed would be a simple database abstraction layer by which events can be appended.

```php
interface EventSourcedAggregateRoot
{
    public static function reconstitute(EventStream $events);
}

class Post extends AggregateRoot implements EventSourcedAggregateRoot
{
    public static function reconstitute(EventStream $history)
    {
        $post = new static($history->getAggregateId());
        foreach ($events as $event) {
            $post->applyThat($event);
        }
        return $post;
    }
}
```

Now the Post aggregate has a method which given a set of events \(or in other words an event stream\) is able to replay the state step by step until it reaches the current state before saving. The next step would be building an adapter of the PostRepository port that will fetch all the published events from the Post aggregate and append them to the data store where all the events are appended. This is what we call an eventstore.

```php
class EventStorePostRepository implements PostRepository
{
    private $eventStore;
    private $projector;

    public function __construct($eventStore, $projector)
    {
        $this->eventStore = $eventStore;
        $this->projector = $projector;
    }

    public function save(Post $post)
    {
        $events = $post->recordedEvents();
        $this->eventStore->append(new EventStream($post->id(), $events));
        $post->clearEvents();
        $this->projector->project($events);
    }
}
```

This is how the implementation of the PostRepository looks like when we use an eventstore to save all the events published by the Post aggregate. Now we need a way to restore an aggregate from its events history. A reconstitute method implemented by the Post aggregate to be used to rebuild a blog post state from triggered events comes in very handy.

```php
class EventStorePostRepository implements PostRepository
{
    public function byId(PostId $id)
    {
        return Post::reconstitute(
            $this->eventStore->getEventsFor($id)
        );
    }
}
```

The eventstore is the work-horse that carries out all the responsibility in regard to saving and restoring eventstreams. Its public API is composed of two simple methods: append and getEvents-From. The former appends an eventstream to the eventstore and the later loads eventstreams to allow aggregate rebuilding. We could use a key-value implementation to store all events

```php
<?php

class EventStore
{
    private $redis;
    private $serializer;

    public function __construct($redis, $serializer)
    {
        $this->redis = $redis;
        $this->serializer = $serializer;
    }

    public function append(EventStream $eventstream)
    {
        foreach ($eventstream as $event) {
            $data = $this->serializer->serialize(
                $event,
                'json'
            );
            $date = (new DateTimeImmutable())->format('YmdHis');
            $this->redis->rpush(
                'events:' . $event->getAggregateId(),
                $this->serializer->serialize([
                    'type' => get_class($event),
                    'created_on' => $date,
                    'data' => $data
                ], 'json')
            );
        }
    }

    public function getEventsFor($id)
    {
        $serializedEvents = $this->redis->lrange(
            'events:' . $id,
            0,
            -1
        );
        $eventStream = [];
        foreach ($serializedEvents as $serializedEvent) {
            $eventData = $this->serializer->deserialize(
                $serializedEvent,
                'array',
                'json'
            );
            $eventStream[] = $this->serializer->deserialize(
                $eventData['data'],
                $eventData['type'],
                'json'
            );
        }
        return new EventStream($id, $eventStream);
    }
}
```

This eventstore implementation is built upon Redis¹¹, a widely used key-value store. The events are appended in a list using the prefix “events:”. In addition, before persisting the events we extract some metadata like the event class or the creation date, as it will come handy later.Obviously, in terms of performance, it is expensive for an aggregate to go over its full event history to reach its final state all of the time. This is especially the case when an eventstream has hundreds or even thousands of events. The best way to overcome this situation is to take a snapshot from the aggregate and replay only the events in the eventstream since the snapshot was taken. A snapshot  is just a simple serialized version of the aggregate state at a given moment. It can be based on the number of events of the aggregate’s eventstream or time-based. With the first approach, a snapshot will be taken every n triggered events \(every 50, 100 or 200 for example\). With the second approach a snapshot will be taken every n seconds. To follow the example, we will use the first way of snapshotting. In the event’s metadata we store an additional field, the version, from which we will start replaying the aggregate history

```php
<?php

class SnapshotRepository
{
    public function byId($id)
    {
        $key = 'snapshots:' . $id;
        $metadata = $this->serializer->unserialize(
            $this->redis->get($key)
        );
        if (null === $metadata) {
            return;
        }
        return new Snapshot(
            $metadata['version'],
            $this->serializer->unserialize(
                $metadata['snapshot']['data'],
                $metadata['snapshot']['type'],
                'json'
            )
        );
    }

    public function save($id, Snapshot $snapshot)
    {
        $key = 'snapshots:' . $id;
        $aggregate = $snapshot->aggregate();
        $snapshot = [
            'version' => $snapshot->version(),
            'snapshot' => [
                'type' => get_class($aggregate),
                'data' => $this->serializer->serialize(
                    $aggregate,
                    'json'
                    )
            ]
        ];
        $this->redis->set($key, $snapshot);
    }
}
```

And now we need to refactor the EventStore class so that it starts using the SnapshotRepository to load the aggregate with acceptable performance times

```php
class EventStorePostRepository implements PostRepository
{
    public function byId(PostId $id)
    {
        $snapshot = $this->snapshotRepository->byId($id);
        if (null === $snapshot) {
            return Post::reconstitute(
                $this->eventStore->getEventsFrom($id)
            );
        }
        $post = $snapshot->aggregate();
        $post->replay(
            $this->eventStore->fromVersion($id, $snapshot->version())
        );
        return $post;
    }
}
```

We just need to take aggregate snapshots periodically. We could do this synchronously or asynchronously by a process responsible for monitoring the eventstore.The following code is a simple example demonstrating the implementation of aggregate snapshotting.

```php
class EventStorePostRepository implements PostRepository
{
    public function save(Post $post)
    {
        $id = $post->id();
        $events = $post->recordedEvents();
        $post->clearEvents();
        $this->eventStore->append(
            new EventStream($id, $events)
        );
        $countOfEvents = $this->eventStore->countEventsFor(
            $id
        );
        $version = $countOfEvents / 100;
        if (!$this->snapshotRepository->has($post->id(), $version)) {
            $this->snapshotRepository->save(
                $id,
                new Snapshot(
                    $post,
                    $version
                )
            );
        }
        $this->projector->project($events);
    }
}
```



