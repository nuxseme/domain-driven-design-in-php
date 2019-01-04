If we want to include some additional behaviour into our Money class, like an add method, it feels natural to check that the input fits any preconditions and maintains any invariance. In our case, we only wish to add monies with the same currency.

如果我们想在Money类中包含一些额外的行为，比如add方法，那么检查输入是否符合任何先决条件并保持任何不变性是很自然的。在我们的情况下，我们只希望增加货币与相同的货币。

```php
class Money
{
// ...

    public function add(Money $money)
    {
        if ($money->currency() !== $this->currency()) {
            throw new \InvalidArgumentException();
        }

        $this->amount += $money->amount();
    }
}
```

If the two currencies do not match, an exception is raised. Otherwise, the amounts are added. How- ever, this code has some undesirable pitfalls. Now imagine we have another method otherMethod.

如果两种货币不匹配，则会引发异常。否则，将添加金额。然而，这段代码有一些不受欢迎的缺陷。现在想象我们有另一种方法

```php
class Banking
{
    public function aMethod()
    {
        $aMoney  =  new  Money(100,  new  Currency('USD'));
        $this->otherMethod($aMoney);
// ...
    }
}
```

Everything is fine until for some reason we start seeing unexpected results in $aMoney. What happens if otherMethod uses our previously defined add method? Maybe you are unaware that add mutates the state of the Money instance. This is what we call a side-effect. You should never mutate arguments, as the client never expects this behaviour.

So, how can we fix this? Simple, by making sure that the Value Object remains immutable we avoid this kind of unexpected problem. A simple solution could be returning a new instance for every potentially mutable operation, like the add method

一切都很好，直到由于某种原因，我们开始看到意外的结果在金钱。如果otherMethod使用我们之前定义的add方法会发生什么?也许您没有意识到add会改变Money实例的状态。这就是我们所说的副作用。您永远不应该改变参数，因为客户机永远不会期望这种行为。

那么，我们如何解决这个问题呢?很简单，通过确保值对象保持不变，我们可以避免这种意外的问题。一个简单的解决方案是为每个潜在的可变操作返回一个新实例，比如add方法



```php
class Money
{
//...

    public function add(Money $money)
    {
        if (!$money->currency()->equals($this->currency())) { throw new \InvalidArgumentException();
        }

        return new self(
            $money->amount() + $this->amount(),
            $this->currency()
        );
    }
}
```

With this simple change, immutability is guaranteed. Each time two Money are added together, a new resulting instance is returned. Other classes can perform any number of changes, without affecting the original copy. Code free of side-effects is easy to understand, easy to test and less error-prone.

通过这个简单的更改，可以保证不变性。每次将两个货币相加，都会返回一个新的结果实例。其他类可以执行任意数量的更改，而不会影响原始副本。无副作用的代码易于理解，易于测试，且不易出错

