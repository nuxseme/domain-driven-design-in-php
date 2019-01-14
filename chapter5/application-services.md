Application services are the middleware between the outside world and the domain logic. The purpose of such a mechanism is to transform commands from the outside world into meaningful domain instructions.

Let’s consider the User signs in into our platform use case. Starting with an outside-in approach, from the delivery mechanism we need to compose the input request for our domain operation. Using a framework like Symfony 2 as the delivery mechanism the code would be something like

```php
class SignInController extends Controller
{
    public function signInAction(Request $request)
    {
        $signInService = new SignInUserService($this->get('user_repository'));

        try {
            $response = $signInService->execute(new SignInUserRequest(
                $request->request->get('email'),
                $request->request->get('password')
            ));
        } catch(UserAlreadyExistsException $e) {
            $this->render('error.html.twig', $response);
        }

        return $this->render('success.html.twig', $response);
    }
}
```

On the domain side, the Application Service that coordinates the logic that fulfils the User signs in use case

```php
class SignInUserService implements Service
{
    private $userRepository;

    public function construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function execute(SignInUserRequest $request)
    {
        $user = $this->userRepository->userOfEmail($request->email);
        if ($user) {
            throw new UserAlreadyExistsException();
        }

        $user = new User(
                $this->userRepository->nextIdentity(),
                $request->email,
                $request->password
            );

        $this->userRepository->persist($user);

        return new SignInUserResponse($user);
    }
}
```

As following the same contract for every service is convenient \(will see that later\), the communi- cation between the delivery mechanism and the domain is carried by data structures called DTOs \(Data Transfer Objects\).





```php
class SignInUserRequest
{
    public $email;
    public $password;

    public function construct($email, $password)
    {
        $this->email = $email;
        $this->password = $password;
    }
}

class SignInUserResponse
{
    public $user;

    public function construct(User $user)
    {
        $this->user = $user;
    }
}
```



