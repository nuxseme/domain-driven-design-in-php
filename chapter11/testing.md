As you are interested in testing the behaviour of the Application Service itself, there is no need to turn it into an Integration Test with complicated setups going against a real database. You are not interested in testing the low-level details. Most of the time a Unit Test will be enough.

由于您对测试应用程序服务本身的行为感兴趣，因此没有必要将其转换为针对真实数据库的复杂设置的集成测试。您对测试底层细节不感兴趣。大多数情况下，单元测试就足够了。

```php
class SignUpUserServiceTest extends \PHPUnit_Framework_TestCase
{
    /**
     * @var \Lw\Domain\Model\User\UserRepository
     */
    private $userRepository;
    /**
     * @var SignUpUserService
     */
    private $SignUpUserService;
    public function setUp()
    {
        $this->userRepository = new InMemoryUserRepository();
        $this->SignUpUserService = new SignUpUserService($this->userRepository);
    }
    /**
     * @test
     * @expectedException \Lw\Domain\Model\User\UserAlreadyExistsException
     */
    public function alreadyExistingEmailShouldThrowAnException()
    {
        $this->executeSignUp();
        $this->executeSignUp();
    }
    private function executeSignUp()
    {
        return $this->SignUpUserService->execute(
            new SignUpUserRequest(
                'user@example.com',
                'password'
            )
        );
    }
    /**
     * @test
     */
    public function afterUserSignUpItShouldBeInTheRepository()
    {
        $user = $this->executeSignUp();
        $this->assertSame(
            $user,
            $this->userRepository->ofId($user->id())
        );
    }
}
```

We’ve used an in-memory implementation for the User Repository. This is what is called a Fake, a fully functional implementation for the repository that will make our test work as a unit. We don’t need to go to the database to test the behaviour of this class. That would make our test slow and fragile.

Checking for Domain Events submission might be interesting too. If creating a user fires a user registered event, assuring it has been triggered might be a good idea.

我们已经为用户存储库使用了内存中的实现。这就是所谓的伪代码，它是存储库的全功能实现，将使我们的测试作为一个单元工作。我们不需要去数据库测试这个类的行为。这将使我们的测试变得缓慢和脆弱。



检查域事件提交可能也很有趣。如果创建一个用户触发一个用户注册的事件，确保它已经被触发可能是一个好主意。

```php
class SignUpUserServiceTest extends \PHPUnit_Framework_TestCase
{
    //...
    /**
     * @test
     */
    public function itShouldPublishUserRegisteredEvent()
    {
        $subscriber = new SpySubscriber();
        $id = DomainEventPublisher::instance()->subscribe($subscriber);
        $user = $this->executeSignUp();
        $userId = $user->id();
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



