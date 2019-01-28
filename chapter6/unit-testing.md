You know already how to publish DomainEvents, but how we can unit test that such publishing happens? How can we really test that UserRegistered is really fired? The easiest way we suggest

is to use a specific EventListener that will work as an Spy⁹ to record if the Domain Event was published. Let’s see an example of the User entity unit test.

```php
use Ddd\Domain\DomainEventPublisher;
use Ddd\Domain\DomainEventSubscriber;

class UserTest extends \PHPUnit_Framework_TestCase
{
//...

    /**
    @test
     */
    public function itShouldPublishUserRegisteredEvent()
    {
        $subscriber = new SpySubscriber();
        $id = DomainEventPublisher::instance()->subscribe($subscriber);

        $userId = new UserId();
        new User($userId, 'valid@email.com', 'password');
        DomainEventPublisher::instance()->unsubscribe($id);

        $this->assertUserRegisteredEventPublished($subscriber, $userId);
    }

    private function assertUserRegisteredEventPublished($subscriber, $userId)
    {
        $this->assertInstanceOf('UserRegistered', $subscriber->domainEvent);
        $this->assertTrue($subscriber->domainEvent->userId()->equals($userId));
    }
}

class SpySubscriber implements DomainEventSubscriber
{
    public $domainEvent;

    public function handle($aDomainEvent)
    {
        $this->domainEvent = $aDomainEvent;
    }

    public function isSubscribedTo($aDomainEvent)
    {
        return true;
    }
}
```

There are some alternatives. You could use a static setter for the DomainEventPublisher or use some reflection framework to detect the call. However, we think this approach is more natural. Last but not least, remember to clean up the spy subscription so it won’t affect the rest of the unit tests execution.

