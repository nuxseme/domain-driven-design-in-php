Given the user authentication example from multiple domain service implementations, it is ex- tremely beneficial to be able to easily test the service. Typically however, testing Template Method implementations can be tricky, as a result we will be using a plain password encryption implementation for testing purposes.

```php
class PlainPasswordEncryption implements PasswordEncryption
{
    public function verify($plainPassword, $hash)
    {
        return $plainPassword === $hash;
    }
}
```

Now we can test all cases in the domain service



```php

class SignInTest extends PHPUnit_Framework_TestCase
{
    private $signIn;
    private $userRepository;

    protected function setUp()
    {
        $this->userRepository = new InMemoryUserRepository();
        $this->signIn = new SignIn(
            $this->userRepository,
            new PlainPasswordEncryption()
        );
    }

    /**
    @test
    @expectedException InvalidArgumentException
     */
    public function itShouldComplainIfTheUserDoesNotExist()
    {
        $this->signIn->execute('test-username', 'test-password');
    }

    /**
    @test
    @expectedException BadCredentialsException
     */
    public function itShouldTellIfTheUserIsFoundButThePasswordDoesNotMatch()
    {
        $this->userRepository->add(
            new User(
                'test-username',
                'test-password'
            )
);

$this->signIn->execute('test-username', 'no-matching-password')
}

    /**
    @test
     */
    public function itShouldTellIfTheUserIsFoundAndMatchesTheProvidedPassword()
    {
        $this->userRepository->add(
            new User(
                'test-username', 'test-password'
            )
        );

        $this->assertInstanceOf( 'Ddd\Domain\Model\User\User',
            $this->signIn->execute('test-username', 'test-password')
        );
    }
}
```



