If we want to include some additional behaviour into our Money class, like an add method, it feels natural to check that the input fits any preconditions and maintains any invariance. In our case, we only wish to add monies with the same currency.

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



