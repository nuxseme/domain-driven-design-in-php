When modeling Events, name them and their properties according to the Ubiquitous Language in the Bounded Context where they originate. If an Event is the result of executing a command operation on an Aggregate, the name is usually derived from the command that was executed. It is important that the Event name reflects the past nature of the occurrence. It is not occurring now. It occurred previously. The best name to choose is the one that reflects that fact.

Let’s consider our user registration feature and the DomainEvent needs to represent that fact. The following code shows a minimal interface for a base DomainEvent.

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

> As rule of thumb:
>
> ```
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
> ```

But, what happens if your Domain experts from the same BC or a different one needs more information? Let’s see the same Domain Event modeled with more information, for example, the email address.

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

Do you remember the gamification example? In order to create the users of the gamification platform, probably called Player, the UserId from the e-commerce Bounded Context is probably enough. But, what happens if the gamification platform has to notify the users by email about being rewarded? In this case, the email address is also mandatory. So, if in the original Domain Event, the email address is included we are done. If that?s not the case, the gamification Bounded Context needs to request such information from the e-commerce one via REST or SOA integration.



> Why not the whole User Entity?
>
>         Should I include the whole User Entity from my Bounded Context in the Domain Event? Our suggestion, don’t. Domain Events are used to communicate a Bounded Context with itself and other Bounded Contexts. That means, what can be a Seller in a C2C e-commerce product catalog Bounded Context, can be an Author of a product review in a product feedback one. Both can share the same id or email, but Seller and Author are different concepts represented different entities from different Bounded Contexts. So, Entities from one Bounded Context have no meaning or a totally different one in the others.



