How could we test this Application Service? As Kent Beck suggests in “TDD by Examples”, test everything that could possibly break. That means the happy and the sad paths. Happy path is the one that does the job right when all the input is valid and all the resources are available. In the registration use case, the email and password are valid, the user does not exist already, the database is available, etc. The sad path is every other case.



```php
class SignInUserServiceTest extends \PHPUnit_Framework_TestCase
{
    /**
    @test
    @expectedException \Ddd\Domain\Model\User\UserAlreadyExistsException
     */
    public function alreadyExistingEmailShouldThrowAnException()
    {
        $service = new SignInUserService(new UserRepository());
        $service->execute('user@example.com',   'password');
        $service->execute('user@example.com',   'password');
    }

    /**
    @test
     */
    public function afterUserSignUpItShouldBeInTheRepository()
    {
        $userRepository = new UserRepository();
        $service = new SignInUserService($userRepository);
        $user = $service->execute('user@example.com', 'password');

        $this->assertSame(
            $user,
            $userRepository->userOfId($user->id())
        );
    }
}
```



