Before going into the detail about Domain Events, let’s see a real example about using Domain Events and how they can help us in our application and our whole Domain.

Let’s consider a simple Application Service that will register a new user. For example, in an e- commerce context. Application Services will be explained in its own chapter, so don’t worry too much about its interface, just focus on the execute method.

```php

class SignInUserService implements ApplicationService
{
    private $userRepository;
    private $userFactory;
    private $userTransformer;

    public function construct( 
        UserRepository $userRepository, 
        UserFactory $userFactory, 
        UserTransformer $userTransformer
    )
    {
        $this->userRepository = $userRepository;
        $this->userFactory = $userFactory;
        $this->userTransformer = $userTransformer;
    }

    /**
    @param SignInUserRequest $request
    @return User
    @throws UserAlreadyExistsException
     */
    public function execute($request = null)
    {
        $email = $request->email();
        $password = $request->password();

        $user = $this->userRepository->userOfEmail($email);
        if (null !== $user) {
            throw new UserAlreadyExistsException();
        }

        $user = $this->userFactory->build(
            $this->userRepository->nextIdentity(),
            $email,
            $password
        );

        $this->userRepository->add($user);
        $this->userTransformer->write($user);
    }
}
```

As shown, the Application Service checks if the user already exists. If not, it creates a new User and adds it to the UserRepository.

Consider now a new requirement. A new user must be notified by email when registered. Without thinking too much, the first approach coming to mind is updating our Application Service to include a piece of code that would do the job. Probably some sort of EmailSender that would be run after the add method. However, let’s consider another approach.

What about firing a UserRegistered event so another component listening to such sort of event can react and send that email? There are some cool benefits about this new approach. First of all, we don’t need to update the code of our Application Service every time a new action must be performed when a new user is registered.

Second, it’s easier to test. The Application Service remains simpler and each time a new action is developed, we just write the tests for the action.

Later in the same e-commerce project, we are told to integrate an open-source gamification platform not written in PHP. Each time a user places a purchase or reviews a product in our e-commerce Bounded Context, she can get badges that can be shown in the e-commerce user profile page or be notified by email. How could we model the problem?

Following the first approach, we would update the Application Service to integrate with the new platform having a similar situation as the confirmation email feature. With the DomainEvent approach, we would create another listener for the UserRegistered event that will connect directly, by REST or SOA, to the gamification platform or even better, would spread it through some messaging system, such as RabbitMQ, so that event can be received by the gamification platform and react accordingly so our e-commerce BC doesn’t know anything about our new gamification BC.



