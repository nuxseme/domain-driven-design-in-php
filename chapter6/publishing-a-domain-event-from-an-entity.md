Carry on with the example of a new user that has been registered in our application, let’s see how the corresponding Domain Event can be published.

```php
class User
{
    protected $userId;
    protected $email; 
    protected $password;

    public function construct(UserId $userId, $email, $password)
    {
        $this->setUserId($userId);
        $this->setEmail($email);
        $this->setPassword($password);

        DomainEventPublisher::instance()->publish(
            new UserRegistered(
                $this->userId
            )
        );
    }

//...
}
```

As seen in the example, when the User is created a new UserRegistered event is published. It’s done in the Entity constructor and not outside because, with this approach, it’s easier to keep our Domain consistent, any client that creates a new User will publish its corresponding event. On the other hand, this makes it a bit more complex to use an infrastructure that needs to create a User Entity without using its constructor. For example, Doctrine uses serialize and unserialize technique that recreates an object without calling its constructor, however, if you have to create your own, this is not going to be as easy as in Doctrine.

In general, constructing an object from plain data such as an array is called hydratation. Let’s see an easy approach to build a new User fetched from a database. First of all, let’s extract the Domain Event publication in a different method applying the Factory Method pattern⁵.

> The template method pattern is a behavioral design pattern that defines the program skeleton of an algorithm in a method, called template method, which defers some steps to subclasses.

```php
class User
{
    protected $userId; 
    protected $email; 
    protected $password;

    public function construct(UserId $userId, $email, $password)
    {
        $this->setUserId($userId);
        $this->setEmail($email);
        $this->setPassword($password);
        $this->publishEvent();

    }

    protected function publishEvent()
    {
        DomainEventPublisher::instance()->publish(
            new UserRegistered(
                $this->userId
            )
        );
    }

//...
}
```

Now, let’s extend our current User with a new infrastructure Entity that will do the job for us. The trick here is make publishEvent a no operation so the Domain Event is not published.

```php
class CustomOrmUser extends User
{
    protected function publishEvent()
    {

    }

    public static function fromRawData($data)
    {
        return new self(
            new UserId($data['user_id']),
            $data['email'],
            $data['password']
        );
    }
}
```

With this approach, when using self-encapsulation as here, be careful when fetching objects from our persistence engine that are invalid because changes in the Domain rules. Another approach without using the parents constructor at all could be:

```php
class CustomOrmUser extends User
{
    public function construct()
    {
    }

    public static function fromRawData($data)
    {
        $user = new self();
        $user->userId = new UserId($data['user_id']);
        $user->email = $data['email'];
        $user->password = $data['password'];

        return $user;
    }
}
```

With this approach, parent constructor is not called and User attributes must be protected. Other alternatives are Reflection, passing flags in the constructor, using some proxy library such as Proxy- Manager⁶ or an ORM such as Doctrine.

