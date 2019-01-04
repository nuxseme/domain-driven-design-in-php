CQRS is a powerful and flexible architecture. There is an added benefit in regard to gathering and saving the d    omain events \(which occurred during an aggregate operation\), giving you a high-level degree of detail of what is going on within your domain. Domain Events are one of the key tactical patterns because of their significance     within the domain, as they describe past occurrences. An ever growing number of events is a smell of the business overlooking insight in the Domain. By using CQRS we gained a highly sophisticated history of all the relevant occurrences at a level that the whole state of the domain model can be expressed just by reproducing domain     events. We just need a tool for storing all those events in a consistent way. This store is called an eventstor  eventstore.

CQRS是一个强大而灵活的架构。收集和保存domain事件\(在聚合操作期间发生的事件\)还有一个额外的好处，可以让您对领域内正在发生的事情有一个高层次的详细了解。领域事件是关键的战术模式之一，因为它们在领域内具有重要意义，因为它们描述了过去发生的事件。越来越多的事件是一种忽略了领域洞察力的商业气息。通过使用CQRS，我们获得了所有相关事件发生的高度复杂的历史记录，在这个级别上，仅仅通过再现域事件就可以表达域模型的整个状态。我们只需要一个以一致的方式存储所有这些事件的工具。这个存储称为eventstor事件存储。

The fundamental idea behind Event Sourcing is to express the state of Aggregates as a linear sequence of events.

With CQRS we partially achieved the following - the Post entity alters its state by using domain events but it is persisted as always, mapping the object to a database table. Event Sourcing takes this a step further. If we were using a database table to store the state of all the blog posts and another to store the state of all the blog post comments and so on, by using Event Sourcing we could use just a single database table. A single append-only database table that would store all the domain events published by all the aggregates within the domain model. Yes, you have read that correctly, a single database table. With this model in mind, tools like object-relational mappers are not needed any more. They would be an overkill for a single database table. The only tool needed would be a simple database abstraction layer by which events can be appended.

事件源背后的基本思想是将聚合状态表示为事件的线性序列。

使用CQRS，我们部分地实现了以下功能:Post实体通过使用域事件更改其状态，但它始终保持不变，将对象映射到数据库表。事件来源则更进一步。如果我们使用一个数据库表存储所有博客文章的状态，而使用另一个数据库表存储所有博客文章评论的状态，等等，通过使用事件源，我们可以只使用一个数据库表。一个仅附加的数据库表，用于存储域模型中所有聚合发布的所有域事件。是的，您已经正确地阅读了，一个数据库表。有了这个模型，就不再需要对象-关系映射器之类的工具了。对于单个数据库表来说，这将是一个巨大的损失。唯一需要的工具是一个简单的数据库抽象层，可以通过它添加事件

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

现在，Post聚合有了一个方法，该方法给出了一组事件\(或者换句话说，一个事件流\)，它能够一步一步地重播状态，直到它达到当前状态，然后再保存。下一步是构建PostRepository端口的适配器，该适配器将从Post聚合中获取所有已发布的事件，并将它们追加到所有事件被追加的数据存储中。这就是我们所说的eventstore

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



当我们使用eventstore保存Post聚合发布的所有事件时，PostRepository的实现是这样的。现在我们需要一种方法来从事件历史中恢复聚合。由Post聚合实现的用于从触发事件重新构建博客状态的重构方法非常方便



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



eventstore是执行保存和恢复eventstreams的所有职责的工作机器。它的公共API由两个简单的方法组成:append和getEvents-From。前者向eventstore追加一个eventstream，而后者加载eventstreams以允许聚合重新构建。我们可以使用键值实现来存储所有事件



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



这个eventstore实现是建立在复述¹¹,一种广泛使用的键值存储。事件使用前缀“events:”追加到列表中。此外，在持久化事件之前，我们提取一些元数据，比如事件类或创建日期，因为它在后面会很有用。显然，就性能而言，对一个聚合来说，遍历其整个事件历史并始终达到其最终状态的代价是昂贵的。当一个eventstream有数百个甚至数千个事件时尤其如此。克服这种情况的最佳方法是从聚合中获取快照，并且只重播自快照获取以来eventstream中的事件。快照只是给定时刻聚合状态的简单序列化版本。它可以基于聚合的事件流的事件数量，也可以基于时间。使用第一种方法，将在每n个触发事件\(例如每50、100或200个事件\)中获取快照。第二种方法是每n秒拍一张快照。为了遵循这个例子，我们将使用第一种快照方式。在事件的元数据中，我们存储了另一个字段version，从该字段开始重播聚合历史记录

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



现在我们需要重构EventStore类，以便它开始使用SnapshotRepository以可接受的性能时间加载聚合

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



我们只需要定期地进行聚合快照。我们可以通过负责监视eventstore的进程来同步或异步地完成此操作。下面的代码是一个简单的示例，演示了聚合快照的实现

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

---

...

