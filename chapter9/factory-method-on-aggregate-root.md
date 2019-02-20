The Factory Method¹ pattern, as defined in the classic Gang of Four, is a creational pattern that…

Defines an interface for creating an object, but leaves the choice of its type to the subclasses, creation being deferred at run-time.

Adding a Factory Method in the Aggregate Root hides the internal implementation details about creating aggregates from any external client. This also moves the responsibility for the integrity of the Aggregate back to the root.

In a Domain model where we have a User and Wish entity, the User acts as the Aggregate Root. There is no Wish without User. The User entity should manage its Aggregates.

The way to move the control of Wish back to the User entity is by placing a Factory Method in the Aggregate Root.

工厂方法¹模式,典型的四人帮,是一个创建的模式……定义用于创建对象的接口，但将其类型的选择留给子类，在运行时延迟创建。在聚合根中添加工厂方法会隐藏关于从任何外部客户端创建聚合的内部实现细节。这也将聚合的完整性责任转移回根节点。在我们拥有用户和Wish实体的域模型中，用户充当聚合根。没有用户就没有愿望。用户实体应该管理其聚合。将Wish控件移回用户实体的方法是在聚合根中放置工厂方法。

```php
class User
{
//...

    public function makeWish(WishId $wishId, $email, $content)
    {
        $wish  =  new  WishEmail(
            $wishId,
            $this->id(),
            $email,
            $content
        );

        DomainEventPublisher::instance()->publish(
            new WishMade($wishId)
        );

        return $wish;
    }
}
```

The client does not need to now the internal details about how the Aggregate Root handles the creation logic at all

客户机现在根本不需要关于聚合根如何处理创建逻辑的内部细节

```php
$wish = $aUser->makeWish(
    $wishRepository->nextIdentity(), 'user@example.com',
    'I want to be free!'
);
```



