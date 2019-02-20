We could return sterile data structures with the information the presentation layer needs. As we’ve seen before, DTOs fit with this scenario. We just need to compose them in the Application Service and return them to the client.

我们可以用表示层需要的信息返回无菌的数据结构。正如我们以前看到的，dto适合这个场景。我们只需要在应用程序服务中组合它们并将它们返回给客户机。

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

UserDTO将从表示层的用户实体公开我们需要的任何只读数据，以避免公开行为。

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

任务完成现在我们可以将参数传递给模板引擎，将它们转换为小部件、标记、子模板，或者对表示端上的数据做任何我们想做的事情。

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

然而，让应用程序服务决定如何构建DTO揭示了另一个限制。由于构建DTO完全依赖于应用程序服务，因此使DTO适应不同的客户机将非常困难。考虑Web控制器上重定向所需的数据和相同用例的REST响应所需的数据。完全不是相同的数据。



让我们允许客户端通过传递特定的DTO汇编器来定义如何构建DTO。

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

现在客户端可以通过传递特定的UserDTOAssembler来定制响应。

