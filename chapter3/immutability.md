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

> static vs. self
>
>  Using static over self can result in undesirable issues when a Value Object inherits from another Value Object.



Due to this immutability we must consider how to handle mutable actions which are commonplace in a stateful context. If we require a state change, we must now instead return a brand new Value Object representation with this change.

If we want to increase the amount of a Money value object for example, we are required to now instead return a new Money instance with the desired modifications. Fortunately, it is relativity simple to abide by this rule, as shown in the example below.

```php
class Money
{
// ...

    public function increaseAmountBy($anAmount)
    {
        return new self(
            $this->amount()  +  $anAmount,
            $this->currency()
        );
    }
}
```



The object returned by increaseAmountBy is different from the one used to invoke the method. This can be observed in the example comparability checks below.

```php
$aMoney  =  new  Money(100,  new  Currency('USD'));
$otherMoney = $aMoney->increaseAmountBy(100); var_dump($aMoney === $otherMoney); // bool(false)
$aMoney = $aMoney->increaseAmountBy(100); var_dump($aMoney === $otherMoney); // bool(false)
```



