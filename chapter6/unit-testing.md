You know already how to publish DomainEvents, but how we can unit test that such publishing happens? How can we really test that UserRegistered is really fired? The easiest way we suggest

您已经知道如何发布DomainEvents，但是我们如何对这样的发布进行单元测试呢?我们如何测试userregistration是否真的被触发了呢?我们建议的最简单的方法

is to use a specific EventListener that will work as an Spy⁹ to record if the Domain Event was published. Let’s see an example of the User entity unit test.

是使用一个特定的事件,将作为一个间谍⁹记录如果域事件发表。让我们看一个用户实体单元测试的例子。

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

有一些替代方案。您可以为DomainEventPublisher使用静态setter，或者使用一些反射框架来检测调用。然而，我们认为这种方法更自然。最后但并非最不重要的是，记住清理spy订阅，这样它就不会影响其他单元测试的执行。

