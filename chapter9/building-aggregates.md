Entities are agnostic to the persistence mechanism. You don’t want to couple and pollute your entities with persistence details.

Take a look at the next Application Service

实体与持久性机制无关。您不希望用持久性细节来耦合和污染实体。

看看下一个应用程序服务

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

与下一个用户实体类似

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

假设我们想使用Doctrine作为基础架构持久性机制。Doctrine要求将id作为普通字符串实例变量，以便正常工作。在我们的实体中，$userId是一个userId值对象。仅仅因为Doctrine向用户实体添加额外的id，就会将持久性机制与域模型耦合起来。



在实体一章中，我们已经看到可以通过在基础结构层中为用户实体创建包装器来使用代理Id解决这个问题

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

正如在应用程序服务中创建DoctrineUser将再次将持久性层与域耦合一样，我们需要使用抽象工厂将创建逻辑从服务中解耦出来。我们可以通过在域中创建接口来实现这一点。

```php
interface UserFactory
{
    public function build(UserId $userId, $email, $password);
}
```

And placing the implementation of it inside our infrastructure layer.

并将其实现放在基础结构层中。

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

解耦后，我们只需要将工厂注入到应用程序服务中

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



