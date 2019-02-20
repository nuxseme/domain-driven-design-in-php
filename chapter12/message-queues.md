RESTful resources is not the only way of enabling integrations between Bounded Contexts. As we will see, messaging middleware enables decoupled integrations between different contexts.

Continuing with the Last Wishes application, we have just implemented a unidirectional relationship between two teams to manage points and badges within their respective contexts. We left important functionality out of scope on purpose: rewarding the user every time they make a wish.

We could go for another Open Host Service with a pull strategy. The Will context will be pulling the Gamification context periodically to get badges on sync \(eg: through an scheduler like Cron\). This solution will impact on the users experience and it will waste a lot of unnecessary resources.

A better approach is to use a messaging middleware. With this solution Contexts could push messages to a middleware \(often a message queue\). Interested parties will be able to subscribe, inspect and consume information on-demand in a decoupled fashion. In order to do this, we need a specialised, shared and common communication language so all the parties can understand the information transmitted. This what is called the Published Language.

RESTful资源并不是实现有限上下文之间集成的唯一方法。正如我们将看到的，消息传递中间件支持不同上下文之间的解耦集成。



继续最后的愿望应用程序，我们刚刚实现了两个团队之间的单向关系，在各自的上下文中管理积分和徽章。我们故意将重要的功能排除在范围之外:每次用户许愿时都给予奖励。



我们可以使用pull策略获得另一个开放主机服务。Will上下文将定期拉动游戏化上下文以获得同步徽章\(例如:通过像Cron这样的调度程序\)。这种解决方案会影响用户体验，会浪费大量不必要的资源。



更好的方法是使用消息传递中间件。使用此解决方案上下文可以将消息推送到中间件\(通常是消息队列\)。感兴趣的各方将能够以一种解耦的方式按需订阅、检查和使用信息。为了做到这一点，我们需要一种专门的、共享的和共同的沟通语言，以便各方都能理解所传递的信息。这就是所谓的出版语言。

> Published Language
>
> Use a well-documented shared language that can express the necessary domain information
>
> as a common medium of communication, translating as necessary into and out of that language.
>
> Eric Evans - Domain-Driven Design, Tackling complexity in the heart of software.
>
> 语言出版
>
>
>
> 使用文档化良好的共享语言来表达必要的域信息
>
>
>
> 作为一种通用的交流媒介，在必要时将其翻译成或翻译成另一种语言。
>
>
>
> 领域驱动的设计，解决软件核心的复杂性。

Thinking about the format of these messages, looking closer at our Domain Model we realise we already have it! Domain Events. There is no need for defining a new communication protocol between Bounded Contexts. We can use Domain Events to define a common language across contexts. Their definition of something that Domain Experts care about just happened just fits perfect with what we are looking after a Published Language.

In our example, we could use RabbitMQ⁹ as a messaging middleware. This is probably one of the most reliable and robust messaging AMQP¹⁰ protocol out there. We will incorparate the widely used PHP libraries php-amqplib¹¹ and RabbitMQBundle¹².

Lets start with the Will context as it is the one which triggers events when the user signs up or when making a wish. As we have already seen in the domain events chapter it is a good idea to store domain events into a persistent mechanism, so we will take it for granted. We need a message publisher to fetch and publish stored domain events from the event store to the messaging middleware. We already did the integration with RabbitMQ in the domain events chapterm so we just need to implement the code in the Gamification Context. We will listen for events triggered by the Will Context. As we are using Symfony on the Gamification side, we can take advantage already with the RabbitMQBundle to make things easier.

We define two message consumers for the User Signed Up and Wish Was Made events.

想想这些消息的格式，仔细看看我们的域模型，我们意识到我们已经有了它!域的事件。没有必要在有界上下文之间定义新的通信协议。我们可以使用域事件跨上下文定义公共语言。他们对领域专家所关心的事情的定义刚好与我们所关注的已发布语言的定义相吻合。



在我们的示例中,我们可以使用RabbitMQ⁹作为消息传递中间件。这可能是其中一个最可靠和健壮的消息AMQP¹⁰协议。我们将incorparate广泛使用PHP库php-amqplib¹¹和RabbitMQBundle¹²。



让我们从Will上下文中开始，因为它是在用户注册或许愿时触发事件的上下文。正如我们在域事件一章中已经看到的，将域事件存储到持久机制中是一个好主意，因此我们将把它视为理所当然。我们需要一个消息发布者来从事件存储区获取并将存储的域事件发布到消息传递中间件。我们已经在domain events chapterm中与RabbitMQ进行了集成，因此我们只需要在游戏化上下文中实现代码。我们将侦听由will上下文触发的事件。当我们在游戏化方面使用Symfony时，我们可以利用RabbitMQBundle使事情变得更简单。



我们为用户注册和Wish Was Made事件定义了两个消息消费者。

```php
namespace AppBundle\Infrastructure\Messaging\PhpAmqpLib;
use Lw\Gamification\Command\SignupCommand;
use OldSound\RabbitMqBundle\RabbitMq\ConsumerInterface;
use PhpAmqpLib\Message\AMQPMessage;
class PhpAmqpLibLastWillUserRegisteredConsumer implements ConsumerInterface
{
    private $commandBus;
    public function __construct($commandBus)
    {
        $this->commandBus = $commandBus;
    }
    public function execute(AMQPMessage $message)
    {
        $type = $message->get('type');
        if ('Lw\Domain\Model\User\UserRegistered' === $type) {
            $event = json_decode($message->body);
            $eventBody = json_decode($event->event_body);
            $this->commandBus->handle(
                new SignupCommand($eventBody->user_id->id)
            );
            return true;
        }
        return false;
    }
}
```

Note that in this case we are only processing messages whose type is Lw\Domain\Model\User\UserRegistered. And the consumer for the User Signed Up event.

注意，在本例中，我们只处理类型为Lw\Domain\Model\User\UserRegistered的消息。以及用户注册事件的使用者。

```php
namespace AppBundle\Infrastructure\Messaging\PhpAmqpLib;
use Lw\Gamification\Command\RewardUserCommand;
use Lw\Gamification\DomainModel\AggregateDoesNotExist;
use OldSound\RabbitMqBundle\RabbitMq\ConsumerInterface;
use PhpAmqpLib\Message\AMQPMessage;
class PhpAmqpLibLastWillWishWasMadeConsumer implements ConsumerInterface
{
    private $commandBus;
    public function __construct($commandBus)
    {
        $this->commandBus = $commandBus;
    }
    public function execute(AMQPMessage $message)
    {
        $type = $message->get('type');
        if ('Lw\Domain\Model\Wish\WishWasMade' === $type) {
            $event = json_decode($message->body);
            $eventBody = json_decode($event->event_body);
            try {
                $points = 5;
                $this->commandBus->handle(
                    new RewardUserCommand(
                        $eventBody->user_id->id,
                        $points
                    )
                );
            } catch (AggregateDoesNotExist $e) {
                // Noop
            }
            return true;
        }
        return false;
    }
}
```

Again, we are only interested in tracking Lw\Domain\Model\Wish\WishWasMade events.

In both cases we use a Command Bus, which is out of the scope of this chapter. We can summarise it as a highway that decouples the Command and Receiver. The when and how a Command is executed is independent from who triggered it.

The Gamification Context uses Tactician¹³ \(and TacticianBundle¹⁴\), a simple command bus that can be extended and adapted to your system.

So now we are almost ready to start consuming events from the Will Context. The only missing piece is to define the RabbitMQBundle configuration in Symfony’s config.yml file

同样，我们只对跟踪Lw\Domain\Model\Wish\WishWasMade事件感兴趣。



在这两种情况下，我们都使用命令总线，这超出了本章的范围。我们可以把它概括为一条将命令和接收器解耦的高速公路。命令执行的时间和方式与触发它的人无关。



游戏化上下文使用战术家¹³\(TacticianBundle¹⁴\),一个简单的命令总线,可以扩展和适应您的系统。



现在我们几乎准备好开始从Will上下文中消费事件了。唯一缺少的部分是在Symfony的配置中定义RabbitMQBundle配置。yml文件

```php
services:
last_will_user_registered_consumer:
class: AppBundle\Infrastructure\Messaging\PhpAmqpLib\PhpAmqpLibLastWillU\
 serRegisteredConsumer
arguments:
- @tactician.commandbus
last_will_wish_was_made_consumer:
class: AppBundle\Infrastructure\Messaging\PhpAmqpLib\PhpAmqpLibLastWillW\
 ishWasMadeConsumer
arguments:
- @tactician.commandbus
old_sound_rabbit_mq:
connections:
default:
host: "%rabbitmq_host%"
port: "%rabbitmq_port%"
user: "%rabbitmq_user%"
password: "%rabbitmq_password%"
vhost: "%rabbitmq_vhost%"
lazy: true
consumers:
last_will_user_registered:
connection: default
callback: last_will_user_registered_consumer
exchange_options:
name: last-will
type: fanout
queue_options:
name: last-will
last_will_wish_was_made:
connection: default
callback: last_will_wish_was_made_consumer
exchange_options:
name: last-will
type: fanout
queue_options:
name: last-will
```

Probably, the most convenient RabbitMQ configuration is the Publish / Subscribe¹⁵\*\* pattern. All messages published by the Will Context will be delivered to all connected consumers. This is called

\*fanout in the RabbitMQ exchange configuration. The exchange consists of an agent being in charge of delivering messages to the corresponding queues.

也许,最方便的RabbitMQ配置发布/订阅¹⁵\* \*模式。Will上下文发布的所有消息都将传递给所有连接的使用者。这就是所谓的



\*在RabbitMQ交换配置中展开。交换器由负责将消息传递到相应队列的代理组成。

`> php app/console rabbitmq:consumer --messages=1000 last_will_user_registered`

`> php app/console rabbitmq:consumer --messages=1000 last_will_wish_was_made`

With those two commands Symfony will execute both consumers and they will start listening for Domain Events. We have specified a limit of 1000 messages to consume as PHP is not the best platform to execute long-running processes. It also might be a good idea to use something like Supervisor¹⁶ to monitor and restart processes periodically.

有了这两个命令，Symfony将同时执行两个使用者，并开始侦听域事件。我们指定了消费1000条消息的限制，因为PHP不是执行长时间运行的流程的最佳平台。它也可能是一个好主意使用类似主管¹⁶定期监控和重新启动进程。

