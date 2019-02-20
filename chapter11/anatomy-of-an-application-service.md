Once we have the data encapsulated in a request, it’s time for the business logic. As Vaughn Vernon says: “Keep Application Services thin, using them only to coordinate tasks on the model”.

The first thing to do is to extract the necessary information from the request, this is the email and password. At a high level then we need if there is an existing user with that email already. If it’s not the case, we then create and add the user to the UserRepository. On the special case of finding a user with the same email, we raise an exception so the client could treat it their own way, displaying an error, retrying or just ignoring it.

一旦我们将数据封装到请求中，就到了业务逻辑的时候了。正如沃恩•弗农\(Vaughn Vernon\)所说:“保持应用程序服务的精简，只使用它们来协调模型上的任务”。



首先要做的是从请求中提取必要的信息，这是电子邮件和密码。在一个较高的层次上，我们需要如果有一个现有的用户已经有电子邮件。如果不是这样，那么我们将创建用户并将其添加到UserRepository中。在寻找具有相同电子邮件的用户的特殊情况下，我们会抛出一个异常，以便客户端可以以自己的方式处理它，显示错误、重试或忽略它。

```php
namespace Lw\Application\Service\User;
use Ddd\Application\Service\ApplicationService;
use Lw\Domain\Model\User\User;
use Lw\Domain\Model\User\UserAlreadyExistsException;
use Lw\Domain\Model\User\UserRepository;
class SignUpUserService
{
    private $userRepository;
    public function __construct(
        UserRepository $userRepository
    ) {
        $this->userRepository = $userRepository;
    }
    public function execute(SignUpUserRequest $request)
    {
        $email = $request->email();
        $password = $request->password();
        $user = $this->userRepository->ofEmail($email);
        if (!$user) {
            throw new UserAlreadyExistsException();
        }
        $this->userRepository->add(
            new User(
                $this->userRepository->nextIdentity(),
                $email,
                $password
            )
        );
    }
}
```

Nice! If you are wondering what is this UserRepository thing doing in the constructor, we’ll see next.

好了!如果您想知道这个UserRepository在构造函数中做什么，我们将在下一节中看到。

> Handling Exceptions
>
> Exceptions raised by application services are a way of communicating unusual cases and flows to the client. Exceptions on this layer are related with business logic \(like not finding a user\), not with implementation details like PDOException, PredisException or DoctrineException.
>
> 应用程序服务引发的异常是一种向客户端传递异常情况和流的方式。这一层的异常与业务逻辑\(比如找不到用户\)相关，而与实现细节\(比如PDOException、predisxception或DoctrineException\)无关。



