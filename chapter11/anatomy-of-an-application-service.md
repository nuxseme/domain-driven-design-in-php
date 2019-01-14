Once we have the data encapsulated in a request, it’s time for the business logic. As Vaughn Vernon says: “Keep Application Services thin, using them only to coordinate tasks on the model”.

The first thing to do is to extract the necessary information from the request, this is the email and password. At a high level then we need if there is an existing user with that email already. If it’s not the case, we then create and add the user to the UserRepository. On the special case of finding a user with the same email, we raise an exception so the client could treat it their own way, displaying an error, retrying or just ignoring it.



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

> Handling Exceptions
>
> Exceptions raised by application services are a way of communicating unusual cases and flows to the client. Exceptions on this layer are related with business logic \(like not finding a user\), not with implementation details like PDOException, PredisException or DoctrineException.





