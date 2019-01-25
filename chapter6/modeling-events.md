When modeling Events, name them and their properties according to the Ubiquitous Language in the Bounded Context where they originate. If an Event is the result of executing a command operation on an Aggregate, the name is usually derived from the command that was executed. It is important that the Event name reflects the past nature of the occurrence. It is not occurring now. It occurred previously. The best name to choose is the one that reflects that fact.

Let’s consider our user registration feature and the DomainEvent needs to represent that fact. The following code shows a minimal interface for a base DomainEvent.

在建模事件时，根据事件起源的有限上下文中普遍存在的语言来命名它们及其属性。如果事件是在聚合上执行命令操作的结果，则该名称通常派生自所执行的命令。重要的是，事件名称要反映事件的过去性质。现在还没有。它发生之前。最好的选择是反映这一事实的名字。

让我们考虑一下用户注册特性，DomainEvent需要表示这个事实。下面的代码显示了基本DomainEvent的最小接口。

```php
interface DomainEvent
{
    /**
    @return \DateTime
     */
    public function occurredOn();
}
```

As seen, the minimum information required is a DateTime in order to know when the event happened.

Let’s model now the new user registration event. The following code could be used in order to model an event representing the fact that a new user has been registered in our application. As explained before, the name should be a verb in the past tense, so UserRegistered is probably a good choice.

如前所述，为了知道事件发生的时间，所需的最小信息是一个DateTime。

现在让我们为新用户注册事件建模。可以使用下面的代码来建模一个事件，该事件表示在我们的应用程序中注册了一个新用户。正如前面所解释的，名称应该是过去时态的动词，所以UserRegistered可能是一个不错的选择。

```php
class UserRegistered implements DomainEvent
{
    private $userId;

    public function construct(UserId $userId)
    {
        $this->userId = $userId;
        $this->occurredOn = new \DateTime();
    }

    public function userId()
    {
        return $this->userId;
    }

    public function occurredOn()
    {
        return $this->occurredOn;
    }
}
```

The minimum amount of information to notify about a new user is possibly her UserId. With this information, any process, command or application service, from the same Bounded Context or a different one, may act to this event.

通知新用户的最小信息量可能是她的用户id。有了这些信息，来自相同或不同有界上下文的任何进程、命令或应用程序服务都可以对该事件进行操作。

> As rule of thumb:
>
> DomainEvents are usually designed as immutable
>
> Constructor will initialize the full state of the DomainEvent
>
> DomainEvents will have getters to access its attributes
>
> Include the identity of the Aggregate that performs the action
>
> Include other Aggregate identities related with the first one
>
> Include parameters that caused the Event if useful
>
> domainevent通常被设计成不可变的
>
> 构造函数将初始化DomainEvent的完整状态
>
> DomainEvents将具有访问其属性的getter
>
> 包括执行操作的聚合的标识
>
> 包括与第一个相关的其他聚合标识
>
> 如果有用，包括导致事件的参数

But, what happens if your Domain experts from the same BC or a different one needs more information? Let’s see the same Domain Event modeled with more information, for example, the email address.

但是，如果来自相同BC或不同BC的域专家需要更多信息，会发生什么情况呢?让我们看看使用更多信息建模的相同域事件，例如电子邮件地址。

```php
class UserRegistered implements DomainEvent
{
    private $userId;
    private $userEmail;

    public function construct(UserId $userId, $userEmail)
    {
        $this->userId = $userId;
        $this->userEmail = $userEmail;
        $this->occurredOn = new \DateTime();
    }

    public function userId()
    {
        return $this->userId;
    }

    public function userEmail()
    {
        return $this->userEmail;
    }

    public  function occurredOn()
    {
    return $this->occurredOn;
    }
}
```

We have added the email address. Adding more information to a DomainEvent can help to improve performance or simplify the integration between different Bounded Contexts. Thinking in other Bounded Context point of view could help modeling events. When a new user is created in the upstream Bounded Context, the downstream one would have to create its own user. Adding the user email, could possibly save a sync request to the upstream Bounded Context in the case the downstream one needs it. Let’s see an example.

我们已经添加了邮箱地址向DomainEvent添加更多信息可以帮助提高性能或简化不同边界上下文之间的集成。从其他有界上下文的角度思考可以帮助建模事件。在上游有界上下文中创建新用户时，下游用户必须创建自己的用户。添加用户电子邮件，可能会将同步请求保存到上游有界上下文，以备下游需要。让我们看一个例子。

Do you remember the gamification example? In order to create the users of the gamification platform, probably called Player, the UserId from the e-commerce Bounded Context is probably enough. But, what happens if the gamification platform has to notify the users by email about being rewarded? In this case, the email address is also mandatory. So, if in the original Domain Event, the email address is included we are done. If that?s not the case, the gamification Bounded Context needs to request such information from the e-commerce one via REST or SOA integration.

你还记得游戏化的例子吗?为了创建游戏化平台\(可能称为Player\)的用户，来自电子商务绑定上下文的用户id可能就足够了。但是，如果游戏化平台必须通过电子邮件通知用户获得奖励，会发生什么情况呢?在这种情况下，电子邮件地址也是必须的。因此，如果在原始域事件中，包含电子邮件地址，我们就完成了。如果呢?但事实并非如此，游戏化的边界环境需要通过REST或SOA集成从电子商务环境请求此类信息。

> Why not the whole User Entity?
>
> Should I include the whole User Entity from my Bounded Context in the Domain Event? Our suggestion, don’t. Domain Events are used to communicate a Bounded Context with itself and other Bounded Contexts. That means, what can be a Seller in a C2C e-commerce product catalog Bounded Context, can be an Author of a product review in a product feedback one. Both can share the same id or email, but Seller and Author are different concepts represented different entities from different Bounded Contexts. So, Entities from one Bounded Context have no meaning or a totally different one in the others.
>
> 我是否应该在领域事件中包含来自我的有界上下文的整个用户实体?我们的建议,不要。领域事件用于与自身和其他有界上下文通信有界上下文。这意味着，可以是C2C电子商务产品目录范围内的卖家，也可以是产品反馈中产品评论的作者。两者都可以共享相同的id或电子邮件，但卖方和作者是不同的概念，表示来自不同边界上下文的不同实体。因此，来自一个有界上下文的实体没有意义，或者来自另一个完全不同的上下文。



