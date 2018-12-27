  


Here comes the tricky part. How do we synchronize the read model with the write model? We

already said we would do it by using domain events captured in a write model transaction. For each

type of domain event captured, a specific projection will be executed. So a one-to-one relationship

between domain events and projections will be set.

Let’s have a look at an example of configuring projections, so that we can get a better idea.

$client=new \Elasticsearch\ClientBuilder::create\(\)-&gt;build\(\);

$projector=new Projector\(\);

$projector-&gt;register\(\[

new Infrastructure\Projection\Elasticsearch\PostWasCreated\($client\),

new Infrastructure\Projection\Elasticsearch\PostWasPublished\($client\),

new Infrastructure\Projection\Elasticsearch\PostWasCategorized\($client\),

new Infrastructure\Projection\Elasticsearch\PostContentWasChanged\($client\),

new Infrastructure\Projection\Elasticsearch\PostTitleWasChanged\($client\),

\]\);

$events =\[

new PostWasCreated\(/\* ... \*/\),

new PostWasPublished\(/\* ... \*/\),

new PostWasCategorized\(/\* ... \*/\),

new PostContentWasChanged\(/\* ... \*/\),

new PostTitleWasChanged\(/\* ... \*/\),

\];

$projector-&gt;project\($event\);

This code is kind of synchronous, but the process can be asynchronous if needed. You could make

your customers aware of this out-of-sync data by placing some alerts.

For the next example, we will use the ampq-lib PHP extension in combination with ReactPHP¹⁰.

// Connect to an AMQP broker

$cnn=new AMQPConnection\(\);

$cnn-&gt;connect\(\);

// Create a channel

$ch=new AMQPChannel\($cnn\);

// Declare a new exchange

$ex=new AMQPExchange\($ch\);

$ex-&gt;setName\('events'\);

$ex-&gt;declare\(\);

// Create an event loop

$loop= React\EventLoop\Factory::create\(\);

// Create a producer that will send any waiting messages every half a second

$producer=new Gos\Component\ReactAMQP\Producer\($ex, $loop, 0.5\);

$serializer= JMS\Serializer\SerializerBuilder::create\(\)-&gt;build\(\);

$projector =newAsyncProjector\($producer, $serializer\);

$events =\[

new PostWasCreated\(/\* ... \*/\),

new PostWasPublished\(/\* ... \*/\),

new PostWasCategorized\(/\* ... \*/\),

new PostContentWasChanged\(/\* ... \*/\),

new PostTitleWasChanged\(/\* ... \*/\),

\];

$projector-&gt;project\($event\);

And the event consumer on the RabbitMQ exchange would be something like



// Connect to an AMQP broker

$cnn=new AMQPConnection\(\);

$cnn-&gt;connect\(\);

// Create a channel

$ch=new AMQPChannel\($cnn\);

// Create a new queue

$queue=new AMQPQueue\($ch\);

$queue-&gt;setName\('events'\);

$queue-&gt;declare\(\);

// Create an event loop

$loop= React\EventLoop\Factory::create\(\);

$serializer= JMS\Serializer\SerializerBuilder::create\(\)-&gt;build\(\);

$client=new \Elasticsearch\ClientBuilder::create\(\)-&gt;build\(\);

$projector=new Projector\(\);

$projector-&gt;register\(\[

new Infrastructure\Projection\Elasticsearch\PostWasCreated\($client\),

new Infrastructure\Projection\Elasticsearch\PostWasPublished\($client\),

new Infrastructure\Projection\Elasticsearch\PostWasCategorized\($client\),

new Infrastructure\Projection\Elasticsearch\PostContentWasChanged\($client\),

new Infrastructure\Projection\Elasticsearch\PostTitleWasChanged\($client\),

\]\);

// Create a consumer

$consumer=new Gos\Component\ReactAMQP\Consumer\($queue, $loop, 0.5, 10\);

// Check for messages every half a second and consume up to 10 at a time.

$consumer-&gt;on\(

'consume',

function\($envelope, $queue\)use\($projector, $serializer\) {

$event = $serializer-&gt;unserialize\($envelope-&gt;getBody\(\),'json'\);

$projector-&gt;project\($event\);

}

\);

$loop-&gt;run\(\);

From now on, it could be as simple as making all the needed repositories consume an instance of

  


the projector and make them invoke the projection process.

class DoctrinePostRepository implementsPostRepository

{

private $em;

private $projector;

public function\_\_construct\(EntityManager $em, Projector $projector\)

{

$this-&gt;em= $em;

$this-&gt;projector= $projector;

}

public function save\(Post$post\)

{

$this-&gt;em-&gt;transactional\(function\(EntityManager $em\) use \($post\) {

$em-&gt;persist\($post\);

foreach\($post-&gt;recordedEvents\(\)as$event\) {

$em-&gt;persist\($event\);

}

}\);

$this-&gt;projector-&gt;project\($post-&gt;recordedEvents\(\)\);

}

public function byId\(PostId$id\)

{

return $this-&gt;em-&gt;find\($id\);

}

}

The Post instance and the recorded events are triggered and persisted in the same transaction.

This ensures that no events are lost, as we will project them to the read model if the transaction

is successful. So, no inconsistencies will exist between the write model and the read model.





