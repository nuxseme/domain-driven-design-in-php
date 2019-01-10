Throughout conversations with domain experts, you will come across concepts in the Ubiquitous Language that cannot be neatly represented as either an Entity or Value.



* A user being able to sign-in to a system by themselves?
* A cart being able to be promoted to an order by itself?



The examples above are two concrete concepts which can not naturally be bound to either an Entity or a Value Object. Further highlighting this oddity, we can attempt to model the behavior as follows

```php
class User
{
    public  function  signIn($aUsername,  $aPassword)
    {
// ...
    }
}



class Cart
{
    public function createOrder()
    {
// ...
    }
}

```

In the case of the first implementation, we are not able to know that the given username and password relate to the invoked-upon user instance. Clearly this operation does not suit this Entity, instead it should be extracted out into a separate class, making its intention explicit.

With this thought in mind we could create a domain service with the sole responsibility to authenticate users.

```php
class SignIn
{
    public  function  execute($aUsername,  $aPassword)
    {
// ...
    }
}
```

Or similarly, in the case of the second example, a domain service specialised in creating orders from a supplied cart.



```php
class CreateOrderFromCart
{
    public function execute(Cart $aCart)
    {
// ...
    }
}
```

A domain service can be defined as an operation that fulfills a domain task and naturally does not fit into either an Entity nor a Value Object. As a concept that represents an operation in the domain, they should be used by clients regardless of its run history. Domain services don’t hold any kind of state by themselves, so domain services are stateless operations.



