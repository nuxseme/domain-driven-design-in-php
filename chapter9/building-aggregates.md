Entities are agnostic to the persistence mechanism. You don’t want to couple and pollute your entities with persistence details.

Take a look at the next Application Service



```php
class SignInUserService
{
    private $userRepository;

    public function construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    /**
     * @param SignInUserRequest $request
     */
    public function execute($request)
    {
        $email = $request->email();
        $password = $request->password();

        $user = $this->userRepository->userOfEmail($email);
        if (null !== $user) {
            throw new UserAlreadyExistsException();
        }

        $this->userRepository->persist(new User(
            $this->userRepository->nextIdentity(),
            $email,
            $password
        ));

        return $user;
    }
}

```

With a User entity like the next one



```php
class User
{
    private $userId; 
    private $email; 
    private $password;

    public function construct(UserId $userId, $email, $password)
    {
//...
    }

//...
}

```

Imagine we want to use Doctrine as our infrastructure persistence mechanism. Doctrine requires having an id as a plain string instance variable in order to work properly. In our entity, $userId is a UserId Value Object. Adding an additional id to our User entity just because of Doctrine would couple our persistence mechanism with our domain model.

We’ve seen in the Entities Chapter that we could solve this problem with a Surrogate Id by creating a wrapper around our User entity in the infrastructure layer

```php
class DoctrineUser extends User
{
    private $surrogateUserId;

    public function construct(UserId $userId, $email, $password)
    {
        parent:: construct($userId, $email, $password);
        $this->surrogateUserId = $userId->id();
    }
}

```

As, creating the DoctrineUser in our application service would couple again the persistence layer with our domain, we need to decouple the creation logic out of the service with an Abstract Factory. We could do this by creating an interface in our Domain.



```php
interface UserFactory
{
    public function build(UserId $userId, $email, $password);
}
```



And placing the implementation of it inside our infrastructure layer.



```php
class DoctrineUserFactory implements BaseUserFactory
{
    public function build(UserId $userId, $email, $password)
    {
        return new DoctrineUser($userId, $email, $password);
    }
}

```

Once decoupled, we only need to inject the Factory into our Application Service



```php
class SignInUserService
{
    private $userRepository;
    private $userFactory;

    public function construct( UserRepository $userRepository, UserFactory $userFactory) {
        $this->userRepository = $userRepository;
        $this->userFactory = $userFactory;
    }

    /**
     * @param SignInUserRequest $request
     */
    public function execute($request)
    {
            //...

        $user = $this->userFactory->build(
            $this->userRepository->nextIdentity(),
            $email,
            $password
        );

        $this->userRepository->persist($user);

        return $user;
    }
}

```



