With all DomainEvents persisted into the database, the only thing remaining to spread the news is pushing them to our favorite messaging system. We personally like RabbitMQ¹⁰, but any other such as ActiveMQ or ZeroMQ will do the job. For integrating with RabbitMQ using PHP, there are not many options, php-amqplib¹¹ will do the work.

First of all, we need a service capable of sending persisted DomainEvents to RabbitMQ. That could be easy, what about querying EventStore for all the events and send each one? Not bad, however, we could push the same DomainEvent more than once. In general, we need to minimize the number of DomainEvents republished. If zero times, even better. In order to do that, we need some sort of component to track what DomainEvents have been already pushed and what are the remaining ones. Last but not least, once we know what DomainEvents we have to push, we send the and keep track of the last one published into our messaging system. Let’s see a possible implementation for this service:

```php
class NotificationService
{
    private $serializer;
    private $eventStore;
    private $publishedMessageTracker;
    private $messageProducer;

    public function construct( 
        EventStore $anEventStore,
        PublishedMessageTracker $aPublishedMessageTracker, 
        MessageProducer $aMessageProducer
    )
    {
        $this->eventStore = $anEventStore;
        $this->publishedMessageTracker = $aPublishedMessageTracker;
        $this->messageProducer = $aMessageProducer;
    }

    /**
     * @return int
     */
    public function publishNotifications($exchangeName)
    {
        $publishedMessageTracker = $this->publishedMessageTracker();
        $notifications = $this->listUnpublishedNotifications(
            $publishedMessageTracker->mostRecentPublishedMessageId($exchangeName)
        );

        if (!$notifications) {
            return  0;
        }

        $messageProducer = $this->messageProducer();
        $messageProducer->open($exchangeName);
        try {
            $publishedMessages  =  0;
            $lastPublishedNotification = null;
            foreach ($notifications as $notification) {
                $lastPublishedNotification = $this->publish(
                    $exchangeName,
                    $notification,
                    $messageProducer
                );
                $publishedMessages++;
            }
        } catch(\Exception $e) {
// Log your error (trigger_error, Monolog, etc.)
        }

        $this->trackMostRecentPublishedMessage(
            $publishedMessageTracker,
            $exchangeName,
            $lastPublishedNotification
        );

        $messageProducer->close($exchangeName);

        return $publishedMessages;
    }

    protected function publishedMessageTracker()
    {
        return $this->publishedMessageTracker;
    }

    /**
     * @return StoredEvent[]
     */
    private function listUnpublishedNotifications($mostRecentPublishedMessageId)
    {
        return $this
            ->eventStore()
            ->allStoredEventsSince($mostRecentPublishedMessageId);
    }

    protected function eventStore()
    {
        return $this->eventStore;
    }

    private function messageProducer()
    {
        return  $this->messageProducer;
    }
    private function publish(
        $exchangeName,
        StoredEvent $notification, 
        MessageProducer $messageProducer
    )
    {
        $messageProducer->send(
            $exchangeName,
            $this->serializer()->serialize($notification, 'json'),
            $notification->typeName(),
            $notification->eventId(),
            $notification->occurredOn()
        );

        return $notification;
    }

    private function serializer()
    {
        if (null === $this->serializer) {
            $this->serializer = SerializerBuilder::create()->build();
        }

        return $this->serializer;
    }

    private function trackMostRecentPublishedMessage( 
        PublishedMessageTracker $publishedMessageTracker,
        $exchangeName,
        $notification
    )
    {
        $publishedMessageTracker->trackMostRecentPublishedMessage($exchangeName,$notification);
    }
}
```

NotificationService depends on three interfaces. We have already seen EventStore, responsible for appending and querying about DomainEvents. The second one is PublishedMessageTracker, responsible for keeping track of pushed messages. The third one is MessageProducer, an interface representing our messaging system.

```php
interface PublishedMessageTracker
{
    /**
     *@param string $exchangeName
     *@return int
     */
    public function mostRecentPublishedMessageId($exchangeName);

    /**
     *@param string $exchangeName
     *@param StoredEvent $notification
     */
    public  function  trackMostRecentPublishedMessage($exchangeName,  $notification);
}
```

mostRecentPublishedMessageId method returns the id of last PublishedMessage, so the process can start from the next one. trackMostRecentPublishedMessage is responsible for tracking what’s the last message sent, in order to be able to republish messages in case you need it. $exchangeName represents what communication channel we are going to use to send out our DomainEvents. Let’s see a Doctrine implementation of PublishedMessageTracker.

```php
class DoctrinePublishedMessageTracker extends   EntityRepository implements PublishedMessageTracker
{
    /**
     *@param $exchangeName
     *@return int
     */
    public function mostRecentPublishedMessageId($exchangeName)
    {
        $messageTracked = $this->findOneByExchangeName($exchangeName);
        if (!$messageTracked) {
            return null;
        }

        return $messageTracked->mostRecentPublishedMessageId();
    }

    /**
     *@param $exchangeName
     *@param StoredEvent $notification
     */
    public  function  trackMostRecentPublishedMessage($exchangeName,  $notification)
    {
        if (!$notification) {
            return;
        }

        $maxId = $notification->eventId();

        $publishedMessage = $this->findOneByExchangeName($exchangeName);
        if (null === $publishedMessage) {
            $publishedMessage = new PublishedMessage(
                $exchangeName,
                $maxId
            );
        }

        $publishedMessage->updateMostRecentPublishedMessageId($maxId);

        $this->getEntityManager()->persist($publishedMessage);
        $this->getEntityManager()->flush($publishedMessage);
    }
}
```

This code is quite straightforward. The only edge case, we have to consider, is when no DomainEvent

has been published already.

> Why an exchange name?
>
> We’ll see this in more detail in the “Integrating Bounded Contexts” chapter. However, when a system is running and a new Bounded Context comes into play, you are interested in resending all the DomainEvents to the new BC. So keeping track of the last DomainEvent published and channel is interesting.

In order to keep track of published DomainEvents, we need an exchange name and a notification id. Check a possible implementation.

```php
class PublishedMessage
{
    private $mostRecentPublishedMessageId;
    private $trackerId;
    private  $exchangeName;

    /**
    @param string $exchangeName
    @param int $aMostRecentPublishedMessageId
     */
    public  function      construct($exchangeName,  $aMostRecentPublishedMessageId)
    {
        $this->mostRecentPublishedMessageId = $aMostRecentPublishedMessageId;
        $this->exchangeName  =  $exchangeName;
    }

    public function mostRecentPublishedMessageId()
    {
        return $this->mostRecentPublishedMessageId;
    }

    public function updateMostRecentPublishedMessageId($maxId)
    {
        $this->mostRecentPublishedMessageId = $maxId;
    }

    public function trackerId()
    {
        return $this->trackerId;
    }
}
```

And its corresponding mapping.

```php
Ddd\Domain\Event\PublishedMessage: type: entity
table: event_published_message_tracker
repositoryClass: Ddd\Infrastructure\Application\Notification\DoctrinePublished\ MessageTracker
id:
trackerId:
column: tracker_id type: integer generator:
strategy: AUTO fields:
mostRecentPublishedMessageId:
column: most_recent_published_message_id type: bigint
exchangeName: type:   string
column: type_name
```

Let’s see now what the MessageProducer interface is used for and its implementation details.

```php
interface MessageProducer
{
    public function open($exchangeName);

    /**
    @param $exchangeName
    @param string $notificationMessage
    @param string $notificationType
    @param int $notificationId
    @param \DateTime $notificationOccurredOn
    @return
     */
    public function send(
        $exchangeName,
        $notificationMessage,
        $notificationType,
        $notificationId,
        \DateTime $notificationOccurredOn);

    public function close($exchangeName);
}
```

Quite easy. The open and close methods open and close a connection with the messaging system. send takes a message body, message name and message id and sends them to our messaging engine whatever it is. Because we have chosen RabbitMQ, we need to implement the connection and sending

```php
abstract class RabbitMqMessaging
{
    protected $connection;
    protected $channel;

    public function construct(AMQPConnection $aConnection)
    {
        $this->connection = $aConnection;
        $this->channel = null;
    }

    private function connect($exchangeName)
    {
        if (null !== $this->channel) {
            return;
        }

        $channel = $this->connection->channel();
        $channel->exchange_declare($exchangeName,  'fanout',  false,  true,  false);
        $channel->queue_declare($exchangeName,  false,  true,  false,  false);
        $channel->queue_bind($exchangeName,  $exchangeName);

        $this->channel = $channel;
    }

    public function open($exchangeName)
    {

    }

    protected function channel($exchangeName)
    {
        $this->connect($exchangeName);

        return $this->channel;
    }

    public function close($exchangeName)
    {
        $this->channel->close();
        $this->connection->close();
    }
}


class RabbitMqMessageProducer extends RabbitMqMessaging implements MessageProducer
{
    /**
    @param $exchangeName
    @param string $notificationMessage
    @param string $notificationType
    @param int $notificationId
    @param \DateTime $notificationOccurredOn
     */
    public function send(
        $exchangeName,
        $notificationMessage,
        $notificationType,
        $notificationId,
        \DateTime $notificationOccurredOn
    )
    {
        $this->channel($exchangeName)->basic_publish(
            new AMQPMessage(
                $notificationMessage, [
                    'type' => $notificationType,
                    'timestamp' => $notificationOccurredOn->getTimestamp(), 'message_id' => $notificationId
                ]
            ),
            $exchangeName
        );
    }
}
```

Now that we have a DomainService to push DomainEvents into a messaging system like RabbitMQ, it’s time to execute them. We need to choose a delivery mechanism to run the service. We personally suggest to create a Symfony Console¹² Command.

```php
class PushNotificationsCommand extends Command
{
    protected function configure()
    {
        $this
            ->setName('domain:events:spread')
            ->setDescription('Notify all domain events via messaging')
            ->addArgument( 'exchange-name',
                InputArgument::OPTIONAL,
                'Exchange name to publish events to', 'my-bc-app'
            );
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $app = $this->getApplication()->getContainer();

        $numberOfNotifications =
            $app['notification_service']
                ->publishNotifications(
                    $input->getArgument('exchange-name')
                );

        $output->writeln( sprintf(
                '<comment>%d</comment> <info>notification(s) sent!</info>',
                $numberOfNotifications
            )
        );
    }
}
```

Following the Silex example, let’s see the definition of the $app\['notification\_service'\] defined in the Silex Pimple Service Container¹³.

```php
//...
$app['event_store']  =  $app->share(function($app)  {
    return  $app['em']->getRepository('Ddd\Domain\Event\StoredEvent');
});

$app['message_tracker']  =  $app->share(function($app)  {
    return  $app['em']->getRepository('Ddd\Domain\Event\PublishedMessage');
});

$app['message_producer']  =  $app->share(function()  {
    return new RabbitMqMessageProducer(
        new  AMQPConnection('localhost',  5672,  'guest',  'guest')
    );
});

$app['notification_service']  =  $app->share(function($app)  {
    return new NotificationService(
        $app['event_store'],
        $app['message_tracker'],
        $app['message_producer']
    );
});
//...
```

PHP is not good for long-running processes because of memory leaking. If you need to have a command running for a long time, taking events and pushing them into RabbitMQ there are some options. You need to guarantee that your process is running and running properly. Sometimes, the process is running but the connection with RabbitMQ gets lost. The process goes into a zombie mode. We personally recommend to limit the amount of work that the worker has to do, 1000 items at a time, and finish the process. Then let tools such as Supervisor¹⁴ rerun your job if it finds that it’s not running.

