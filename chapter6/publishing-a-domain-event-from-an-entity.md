Carry on with the example of a new user that has been registered in our application, let’s see how the corresponding Domain Event can be published.

继续以在应用程序中注册的新用户为例，让我们看看如何发布相应的域事件。

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

如示例中所示，当用户创建时，将发布一个新的userregistration事件。它是在实体构造函数中完成的，而不是在外部完成的，因为使用这种方法，更容易保持域的一致性，任何创建新用户的客户机都将发布其相应的事件。另一方面，这使得使用需要在不使用构造函数的情况下创建用户实体的基础设施变得更加复杂。例如，Doctrine使用序列化和非序列化技术，在不调用对象构造函数的情况下重新创建对象，但是，如果您必须创建自己的对象，这就不像Doctrine中那么容易了。

In general, constructing an object from plain data such as an array is called hydratation. Let’s see an easy approach to build a new User fetched from a database. First of all, let’s extract the Domain Event publication in a different method applying the Factory Method pattern⁵.

通常，从普通数据\(如数组\)构造对象称为水合作用。让我们看一种从数据库获取新用户的简单方法。首先,让我们来提取域事件发表在一个不同的方法应用⁵工厂方法模式。

> The template method pattern is a behavioral design pattern that defines the program skeleton of an algorithm in a method, called template method, which defers some steps to subclasses.
>
> template method模式是一种行为设计模式，它在一个名为template method的方法中定义算法的程序框架，该方法将一些步骤推迟到子类中。

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

现在，让我们使用一个新的基础结构实体来扩展我们的当前用户，这个实体将为我们完成这项工作。这里的技巧是使publishEvent成为no操作，这样域事件就不会被发布。

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

使用这种方法，当使用这里的自封装时，从持久性引擎中获取由于域规则更改而无效的对象时要小心。另一种完全不使用父构造函数的方法是:

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

使用这种方法，不调用父构造函数，必须保护用户属性。其他选择反射,通过flags在构造函数中,使用一些代理库比如Proxy- Manager或者ORM。

