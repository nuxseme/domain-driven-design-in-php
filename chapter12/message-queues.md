RESTful resources is not the only way of enabling integrations between Bounded Contexts. As we will see, messaging middleware enables decoupled integrations between different contexts.

Continuing with the Last Wishes application, we have just implemented a unidirectional relationship between two teams to manage points and badges within their respective contexts. We left important functionality out of scope on purpose: rewarding the user every time they make a wish.

We could go for another Open Host Service with a pull strategy. The Will context will be pulling the Gamification context periodically to get badges on sync \(eg: through an scheduler like Cron\). This solution will impact on the users experience and it will waste a lot of unnecessary resources.





A better approach is to use a messaging middleware. With this solution Contexts could push messages to a middleware \(often a message queue\). Interested parties will be able to subscribe, inspect and consume information on-demand in a decoupled fashion. In order to do this, we need a specialised, shared and common communication language so all the parties can understand the information transmitted. This what is called the Published Language.



> Published Language
>
> Use a well-documented shared language that can express the necessary domain information
>
> as a common medium of communication, translating as necessary into and out of that language.
>
> Eric Evans - Domain-Driven Design, Tackling complexity in the heart of software.





Thinking about the format of these messages, looking closer at our Domain Model we realise we already have it! Domain Events. There is no need for defining a new communication protocol between Bounded Contexts. We can use Domain Events to define a common language across contexts. Their definition of something that Domain Experts care about just happened just fits perfect with what we are looking after a Published Language.

In our example, we could use RabbitMQ⁹ as a messaging middleware. This is probably one of the most reliable and robust messaging AMQP¹⁰ protocol out there. We will incorparate the widely used PHP libraries php-amqplib¹¹ and RabbitMQBundle¹².

Lets start with the Will context as it is the one which triggers events when the user signs up or when making a wish. As we have already seen in the domain events chapter it is a good idea to store domain events into a persistent mechanism, so we will take it for granted. We need a message publisher to fetch and publish stored domain events from the event store to the messaging middleware. We already did the integration with RabbitMQ in the domain events chapterm so we just need to implement the code in the Gamification Context. We will listen for events triggered by the Will Context. As we are using Symfony on the Gamification side, we can take advantage already with the RabbitMQBundle to make things easier.

We define two message consumers for the User Signed Up and Wish Was Made events.



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



`> php app/console rabbitmq:consumer --messages=1000 last_will_user_registered`

`> php app/console rabbitmq:consumer --messages=1000 last_will_wish_was_made`



With those two commands Symfony will execute both consumers and they will start listening for Domain Events. We have specified a limit of 1000 messages to consume as PHP is not the best platform to execute long-running processes. It also might be a good idea to use something like Supervisor¹⁶ to monitor and restart processes periodically.







