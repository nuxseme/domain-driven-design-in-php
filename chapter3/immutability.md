This is one of the most important aspects of a Value Object to grasp. Object values should not be able to be altered over their lifetime. Because of this immutability, Value Objects are easy to reason, test and are free of undesired/unexpected side-effects.

As such, Value Objects should be created through their constructor. In order to build one, you usually pass the required primitive types or other value objects through this constructor. Value Objects are always in a valid state, that is why we create them in a single atomic step. Empty constructors with multiple setters and getters move the creation responsibility to the client, resulting in the Anemic Domain Model⁶, which is considered an anti-pattern.

It is also good to point out that it is not recommended to hold references to entities in your Value Objects. Entities are mutable, and as such this could lead to undesirable side-effects occurring in the Value Object.

In languages with method overloading⁷ such as Java, you can create multiple constructors with the same name. Each of these constructors are provided with different options to build the same type of resulting object. In PHP, we are able to provide a similar capability by way of factory methods⁸.



这是要掌握的值对象最重要的方面之一。对象值在其生命周期内不应被更改。由于这种不变性，值对象很容易推理、测试，并且没有不希望的/意外的副作用。

因此，值对象应该通过其构造函数创建。为了构建一个，您通常通过这个构造函数传递所需的基本类型或其他值对象。值对象总是处于有效状态，这就是为什么我们在单个原子步骤中创建它们。具有多个setter和getter的空构造函数将创建责任转移到客户端，导致贫血域模型，它被认为是一个反模式。

还需要指出的是，不建议在值对象中保存对实体的引用。实体是可变的，因此这可能导致在值对象中出现不希望出现的副作用。

在方法重载的语言中，比如Java，你可以用相同的名字创建多个构造函数。每个构造函数都提供了不同的选项来构建相同类型的结果对象。在PHP中，我们可以通过工厂方法提供类似的功能。



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

通过使用self关键字，我们不会将代码与类名结合起来。因此，对类名或名称空间的更改不会影响这些工厂方法。这个小的实现细节有助于以后重构代码。



> static vs. self
>
> Using static over self can result in undesirable issues when a Value Object inherits from another Value Object.
>
> 当一个值对象从另一个值对象继承时，使用静态覆盖self可能会导致不希望出现的问题

Due to this immutability we must consider how to handle mutable actions which are commonplace in a stateful context. If we require a state change, we must now instead return a brand new Value Object representation with this change.

If we want to increase the amount of a Money value object for example, we are required to now instead return a new Money instance with the desired modifications. Fortunately, it is relativity simple to abide by this rule, as shown in the example below.



由于这种不可变性，我们必须考虑如何处理在有状态上下文中常见的可变操作。如果我们需要更改状态，那么现在必须返回一个全新的值对象表示形式。

例如，如果我们想增加货币值对象的数量，我们现在需要返回一个新的货币实例，并进行所需的修改。幸运的是，遵守这个规则是相对简单的，如下面的例子所示。



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

increaseAmountBy返回的对象与用于调用该方法的对象不同。这可以在下面的示例可比性检查中观察到。

```php
$aMoney  =  new  Money(100,  new  Currency('USD'));
$otherMoney = $aMoney->increaseAmountBy(100); var_dump($aMoney === $otherMoney); // bool(false)
$aMoney = $aMoney->increaseAmountBy(100); var_dump($aMoney === $otherMoney); // bool(false)
```



