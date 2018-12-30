Here comes the tricky part. How do we synchronize the read model with the write model? We already said we would do it by using domain events captured in a write model transaction. For each type of domain event captured, a specific projection will be executed. So a one-to-one relationship between domain events and projections will be set. Let’s have a look at an example of configuring projections, so that we can get a better idea.

```php
$client = new \Elasticsearch\ClientBuilder::create()->build();
$projector = new Projector();
$projector->register([
    new Infrastructure\Projection\Elasticsearch\PostWasCreated($client),
    new Infrastructure\Projection\Elasticsearch\PostWasPublished($client),
    new Infrastructure\Projection\Elasticsearch\PostWasCategorized($client),
    new Infrastructure\Projection\Elasticsearch\PostContentWasChanged($client),
    new Infrastructure\Projection\Elasticsearch\PostTitleWasChanged($client),
]);
$events = [
    new PostWasCreated(/* ... */),
    new PostWasPublished(/* ... */),
    new PostWasCategorized(/* ... */),
    new PostContentWasChanged(/* ... */),
    new PostTitleWasChanged(/* ... */),
];
$projector->project($event);
```

This code is kind of synchronous, but the process can be asynchronous if needed. You could make your customers aware of this out-of-sync data by placing some alerts.For the next example, we will use the ampq-lib PHP extension in combination with ReactPHP¹⁰.

```php
// Connect to an AMQP broker
$cnn = new AMQPConnection();
$cnn->connect();
// Create a channel
$ch = new AMQPChannel($cnn);
// Declare a new exchange
$ex = new AMQPExchange($ch);
$ex->setName('events');
$ex->declare();
// Create an event loop
$loop = React\EventLoop\Factory::create();
// Create a producer that will send any waiting messages every half a second
$producer   = new Gos\Component\ReactAMQP\Producer($ex, $loop, 0.5);
$serializer = JMS\Serializer\SerializerBuilder::create()->build();
$projector  = new AsyncProjector($producer, $serializer);
$events     = [
    new PostWasCreated(/* ... */),
    new PostWasPublished(/* ... */),
    new PostWasCategorized(/* ... */),
    new PostContentWasChanged(/* ... */),
    new PostTitleWasChanged(/* ... */),
];
$projector->project($event);
```

And the event consumer on the RabbitMQ exchange would be something like

```php
// Connect to an AMQP broker
$cnn = new AMQPConnection();
$cnn->connect();
// Create a channel
$ch = new AMQPChannel($cnn);
// Create a new queue
$queue = new AMQPQueue($ch);
$queue->setName('events');
$queue->declare();
// Create an event loop
$loop       = React\EventLoop\Factory::create();
$serializer = JMS\Serializer\SerializerBuilder::create()->build();
$client     = new \Elasticsearch\ClientBuilder::create()->build();
$projector = new Projector();
$projector->register([
    new Infrastructure\Projection\Elasticsearch\PostWasCreated($client),
    new Infrastructure\Projection\Elasticsearch\PostWasPublished($client),
    new Infrastructure\Projection\Elasticsearch\PostWasCategorized($client),
    new Infrastructure\Projection\Elasticsearch\PostContentWasChanged($client),
    new Infrastructure\Projection\Elasticsearch\PostTitleWasChanged($client),
]);
// Create a consumer
$consumer = new Gos\Component\ReactAMQP\Consumer($queue, $loop, 0.5, 10);
// Check for messages every half a second and consume up to 10 at a time.
$consumer->on(
    'consume',
    function ($envelope, $queue) use ($projector, $serializer) {
        $event = $serializer->unserialize($envelope->getBody(), 'json');
        $projector->project($event);
    }
);
$loop->run();
```

From now on, it could be as simple as making all the needed repositories consume an instance of the projector and make them invoke the projection process.

```php
class DoctrinePostRepository implements PostRepository
{
    private $em;
    private $projector;

    public function __construct(EntityManager $em, Projector $projector)
    {
        $this->em        = $em;
        $this->projector = $projector;
    }

    public function save(Post $post)
    {
        $this->em->transactional(function (EntityManager $em) use ($post) {
            $em->persist($post);
            foreach ($post->recordedEvents() as $event) {
                $em->persist($event);
            }
        });
        $this->projector->project($post->recordedEvents());
    }

    public function byId(PostId $id)
    {
        return $this->em->find($id);
    }
}
```

The Post instance and the recorded events are triggered and persisted in the same transaction. This ensures that no events are lost, as we will project them to the read model if the transaction is successful. So, no inconsistencies will exist between the write model and the read model.

