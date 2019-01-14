As you are interested in testing the behaviour of the Application Service itself, there is no need to turn it into an Integration Test with complicated setups going against a real database. You are not interested in testing the low-level details. Most of the time a Unit Test will be enough.

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



