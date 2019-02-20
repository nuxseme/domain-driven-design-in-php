With all DomainEvents persisted into the database, the only thing remaining to spread the news is pushing them to our favorite messaging system. We personally like RabbitMQ¹⁰, but any other such as ActiveMQ or ZeroMQ will do the job. For integrating with RabbitMQ using PHP, there are not many options, php-amqplib¹¹ will do the work.

将所有DomainEvents持久化到数据库中后，惟一剩下的事情就是将它们推送到我们最喜欢的消息传递系统。我们个人喜欢RabbitMQ¹⁰,但其他如ActiveMQ或ZeroMQ将做这项工作。RabbitMQ结合使用PHP,没有很多选择,php-amqplib¹¹将做这项工作。

First of all, we need a service capable of sending persisted DomainEvents to RabbitMQ. That could be easy, what about querying EventStore for all the events and send each one? Not bad, however, we could push the same DomainEvent more than once. In general, we need to minimize the number of DomainEvents republished. If zero times, even better. In order to do that, we need some sort of component to track what DomainEvents have been already pushed and what are the remaining ones. Last but not least, once we know what DomainEvents we have to push, we send the and keep track of the last one published into our messaging system. Let’s see a possible implementation for this service:

首先，我们需要一个能够将持久的DomainEvents发送到RabbitMQ的服务。这很简单，那么查询所有事件的EventStore并发送每个事件如何?不过，还不错，我们可以多次推送相同的DomainEvent。通常，我们需要最小化重新发布的DomainEvents的数量。如果是零倍，那就更好了。为了做到这一点，我们需要某种组件来跟踪哪些DomainEvents已经推送，以及哪些是剩余的。最后但并非最不重要的是，一旦我们知道需要推送什么DomainEvents，我们就会向消息传递系统发送并跟踪最后一个发布的domainevent。让我们看看这个服务可能的实现:

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

NotificationService依赖于三个接口。我们已经看到了EventStore，它负责添加和查询关于DomainEvents的信息。第二个是PublishedMessageTracker，负责跟踪推送的消息。第三个是MessageProducer，它是一个表示消息传递系统的接口。

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

mostRecentPublishedMessageId方法返回上一个PublishedMessage的id，因此流程可以从下一个开始。trackMostRecentPublishedMessage负责跟踪最后发送的消息，以便在需要时重新发布消息。$exchangeName表示我们将使用什么通信通道发送我们的domainevent。让我们看看PublishedMessageTracker的原则实现。

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

这段代码非常简单。我们必须考虑的唯一边缘情况是没有DomainEvent的情况

> Why an exchange name?
>
> We’ll see this in more detail in the “Integrating Bounded Contexts” chapter. However, when a system is running and a new Bounded Context comes into play, you are interested in resending all the DomainEvents to the new BC. So keeping track of the last DomainEvent published and channel is interesting.
>
> 我们将在“集成有界上下文”一章中更详细地看到这一点。然而，当一个系统正在运行并且一个新的有界上下文开始发挥作用时，您有兴趣将所有DomainEvents重新发送到新的BC。因此跟踪最近发布的DomainEvent和channel是很有趣的。

In order to keep track of published DomainEvents, we need an exchange name and a notification id. Check a possible implementation.

为了跟踪已发布的DomainEvents，我们需要一个交换名称和一个通知id。

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

现在让我们看看MessageProducer接口的用途及其实现细节。

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

相当容易。打开和关闭方法打开和关闭与消息传递系统的连接。send接收消息主体、消息名称和消息id，并将它们发送到我们的消息传递引擎，不管它是什么。因为我们选择了RabbitMQ，所以需要实现连接和发送

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

现在我们有了一个DomainService，可以将DomainEvents推送到RabbitMQ之类的消息传递系统中，现在可以执行它们了。我们需要选择一种交付机制来运行服务。我们个人建议创建一个Symfony控制台¹²命令。

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

Silex的例子,让我们看看美元的定义应用程序\(“notification\_service”\)中定义的 Silex Pimple Service Container

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

由于内存泄漏，PHP不适合长时间运行的进程。如果您需要命令长时间运行、获取事件并将其推入RabbitMQ，那么有一些选项。您需要确保流程正在正常运行。有时，进程正在运行，但是与RabbitMQ的连接丢失了。进程进入僵死模式。我们个人建议限制工人的工作量，一次完成1000个项目，并完成这个过程。然后让工具,如主管¹⁴重新运行你的工作如果它发现它不运行。

