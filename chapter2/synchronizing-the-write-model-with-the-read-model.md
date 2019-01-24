Here comes the tricky part. How do we synchronize the read model with the write model? We already said we would do it by using domain events captured in a write model transaction. For each type of domain event captured, a specific projection will be executed. So a one-to-one relationship between domain events and projections will be set. Let’s have a look at an example of configuring projections, so that we can get a better idea.

接下来是棘手的部分。我们如何同步读模型和写模型?我们已经说过，我们将通过使用在写模型事务中捕获的域事件来实现。对于捕获的每种类型的域事件，将执行特定的投影。因此，将设置域事件和投影之间的一对一关系。让我们看一个配置投影的示例，以便更好地理解。

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

这段代码是同步的，但是如果需要，这个过程可以是异步的。您可以通过放置一些警告来使您的客户意识到这种不同步的数据。在下一个示例中，我们将结合使用ampq-lib PHP扩展和react tphp

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

RabbitMQ交换上的事件消费者应该是这样的

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

从现在起，可以简单地让所有需要的存储库使用投影的一个实例，并让它们调用投影流程

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

Post实例和记录的事件被触发并保存在同一个事务中。这确保不会丢失事件，因为如果事务成功，我们将把它们投射到read模型。因此，写模型和读模型之间不存在不一致性。

---

1. 读写模型同步？？



