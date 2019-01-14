The Factory Method¹ pattern, as defined in the classic Gang of Four, is a creational pattern that…



Defines an interface for creating an object, but leaves the choice of its type to the subclasses, creation being deferred at run-time.



Adding a Factory Method in the Aggregate Root hides the internal implementation details about creating aggregates from any external client. This also moves the responsibility for the integrity of the Aggregate back to the root.

In a Domain model where we have a User and Wish entity, the User acts as the Aggregate Root. There is no Wish without User. The User entity should manage its Aggregates.

The way to move the control of Wish back to the User entity is by placing a Factory Method in the Aggregate Root.

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

```php
$wish = $aUser->makeWish(
    $wishRepository->nextIdentity(), 'user@example.com',
    'I want to be free!'
);

```







