Before going into the detail about Domain Events, let’s see a real example about using Domain Events and how they can help us in our application and our whole Domain.

Let’s consider a simple Application Service that will register a new user. For example, in an e- commerce context. Application Services will be explained in its own chapter, so don’t worry too much about its interface, just focus on the execute method.



在详细讨论域事件之前，让我们看一个关于使用域事件的真实示例，以及它们如何在应用程序和整个域中帮助我们。

让我们考虑一个注册新用户的简单应用程序服务。例如，在电子商务环境中。应用程序服务将在它自己的章节中进行解释，所以不要太担心它的接口，只需关注执行方法即可。

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

如图所示，应用程序服务检查用户是否已经存在。如果不是，则创建一个新用户并将其添加到UserRepository中。

Consider now a new requirement. A new user must be notified by email when registered. Without thinking too much, the first approach coming to mind is updating our Application Service to include a piece of code that would do the job. Probably some sort of EmailSender that would be run after the add method. However, let’s consider another approach.

现在考虑一个新的需求。新用户注册时必须通过电子邮件通知。不用想太多，首先想到的方法是更新我们的应用程序服务，使其包含一段可以完成这项工作的代码。可能是某种在add方法之后运行的EmailSender。但是，让我们考虑另一种方法。

What about firing a UserRegistered event so another component listening to such sort of event can react and send that email? There are some cool benefits about this new approach. First of all, we don’t need to update the code of our Application Service every time a new action must be performed when a new user is registered.

如果触发一个用户注册的事件，以便另一个组件监听此类事件，从而能够响应并发送该电子邮件，该怎么办?这种新方法有一些很酷的好处。首先，我们不需要在注册新用户时每次必须执行新操作时都更新应用程序服务的代码。

Second, it’s easier to test. The Application Service remains simpler and each time a new action is developed, we just write the tests for the action.

其次，它更容易测试。应用程序服务仍然比较简单，每次开发一个新操作时，我们只编写该操作的测试。

Later in the same e-commerce project, we are told to integrate an open-source gamification platform not written in PHP. Each time a user places a purchase or reviews a product in our e-commerce Bounded Context, she can get badges that can be shown in the e-commerce user profile page or be notified by email. How could we model the problem?

后来在同一个电子商务项目中，我们被告知要集成一个非PHP编写的开源游戏化平台。每次用户在我们的电子商务绑定上下文中放置购买或评论产品时，她都可以获得可以显示在电子商务用户配置文件页面或通过电子邮件通知的徽章。我们如何建模这个问题?

Following the first approach, we would update the Application Service to integrate with the new platform having a similar situation as the confirmation email feature. With the DomainEvent approach, we would create another listener for the UserRegistered event that will connect directly, by REST or SOA, to the gamification platform or even better, would spread it through some messaging system, such as RabbitMQ, so that event can be received by the gamification platform and react accordingly so our e-commerce BC doesn’t know anything about our new gamification BC.

按照第一种方法，我们将更新应用程序服务，以与与确认电子邮件功能类似的新平台集成。DomainEvent方法中,我们将创建另一个UserRegistered事件侦听器直接连接,通过休息或SOA,游戏化平台甚至更好,会传播通过某些消息传递系统,如RabbitMQ,事件可以通过游戏化平台收到和做出相应的反应我们的电子商务公元前公元前不知道任何关于我们的新游戏化。

