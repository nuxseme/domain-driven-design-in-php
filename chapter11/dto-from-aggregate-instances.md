We could return sterile data structures with the information the presentation layer needs. As we’ve seen before, DTOs fit with this scenario. We just need to compose them in the Application Service and return them to the client.



```php
class UserDTO
{
    private $email;
//...
    public function __construct(User $user)
    {
        $this->email = $user->email();
//...
    }
    public function email()
    {
        return $this->email;
    }
}
```



The UserDTO will expose whatever read-only data we need from the User entity on the presentation layer avoiding exposing behaviour.



```php
class SignUpUserService
{
    public function execute(SignUpUserRequest $request)
    {
        // ...
        $user = // ...
        return new UserDTO($user);
    }
}
```







Mission accomplished. Now we could pass parameters to the template engine, transform them into widgets, tags, subtemplates or do whatever we want with the data on the presentation side.



```php
$app->match('/signup', function (Request $request) use ($app) {
    /**
     * @var UserDTO $user
     */
    $userDto = $app['sign_up_user_application_service']->execute(
        new SignUpUserRequest(
            $request->get('email'),
            $request->get('password')
        )
    );
//...
});
```





However, letting the Application Service decide how to build the DTO reveals another limitation. As building the DTO depends exclusively on the Application Service, adapting the DTO to different clients will be very difficult. Consider the data needed for a redirect on a Web Controller and the data needed for a REST response for the same use case. Not the same data at all.

Let’s allow the client to define how to built the DTO by passing a specific DTO assembler.



```php
class SignUpUserService
{
    private $userDtoAssembler;
    public function __construct(
        UserRepository $userRepository,
        UserDTOAssembler $userDtoAssembler
    )
    {
        $this->userRepository = $userRepository;
        $this->userDtoAssembler = $userDtoAssembler;
    }
    public function execute(SignUpUserRequest $request)
    {
        // ...
        $user = // ...
        return $this->userDtoAssembler->assemble($user);
    }
}
```



Now the client can customise the response by passing a specific UserDTOAssembler.



