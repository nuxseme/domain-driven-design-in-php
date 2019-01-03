This is one of the most important aspects of a Value Object to grasp. Object values should not be able to be altered over their lifetime. Because of this immutability, Value Objects are easy to reason, test and are free of undesired/unexpected side-effects.

As such, Value Objects should be created through their constructor. In order to build one, you usually pass the required primitive types or other value objects through this constructor. Value Objects are always in a valid state, that is why we create them in a single atomic step. Empty constructors with multiple setters and getters move the creation responsibility to the client, resulting in the Anemic Domain Model⁶, which is considered an anti-pattern.

It is also good to point out that it is not recommended to hold references to entities in your Value Objects. Entities are mutable, and as such this could lead to undesirable side-effects occurring in the Value Object.

In languages with method overloading⁷ such as Java, you can create multiple constructors with the same name. Each of these constructors are provided with different options to build the same type of resulting object. In PHP, we are able to provide a similar capability by way of factory methods⁸.

In our Money object we could add some useful factory methods, such as:



```php
class Money
{
// ...

    public static function fromMoney(Money $aMoney)
    {
        return new self(
            $aMoney->amount(),
            $aMoney->currency()
        );
    }

    public static function ofCurrency(Currency $aCurrency)
    {
        return  new  self(0,  $aCurrency);
    }
}
```



By using the self keyword we do not couple the code with the class name. As such, a change to the class name or namespace will not effect these factory methods. This small implementation detail aids when refactoring the code at a later date.



