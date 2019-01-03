As discussed at the beginning of the chapter, two Value Objects are equal if the content they measure, quantify, or describe is the same.

For example, conceptualise two Money objects representing 1 USD. Can we consider them equal? In the ‘real’ world are two coins of 1 USD valued the same? Of course they are. Directing our attention back to the code, the Value Objects in question refers to separate instances of Money. However, we can consider them to both represent the same value, so in-turn they are equal.

In regard to PHP, it is common place to compare two Value Objects using the == operator. Examining the PHP documentations⁹ definition of the operator highlights an interesting behaviour.

When using the comparison operator \(==\), object variables are compared in a simple manner, namely: Two object instances are equal if they have the same attributes and values, and are instances of the same class.

This behaviour works in agreement to our formal definition of a Value Object. However, as an exact class match predicate is present, you should be wary when handling sub-typed Value Objects.

With this in mind, the even stricter === operator unfortunately does not help us.

When using the identity operator \(===\), object variables are identical if and only if they refer to the same instance of the same class.

The following example should help confirm these subtle differences.

```php
$a   =   new Currency('USD');
$b   =   new Currency('USD');

var_dump($a == $b); // bool(true)
var_dump($a === $b); // bool(false)
$c = new Currency('EUR'); var_dump($a == $c);	//  bool(false)
var_dump($a === $c); // bool(false)

```

With this in mind a solution is to implement a conventional equals method in each Value Object. This method is tasked with checking the type and equality of its composite attributes. Abstract data type comparability is easy to implement using PHP’s built-in type hinting. On the other hand you can also use the get\_class\(\) function to aid in the comparability check if necessary. The language however, is unable to decipher what equality truly means in your domain concept, meaning it is your responsibility to provide the answer.

In order to compare Currency objects, we just need to compare both their associated ISO codes are the same. The === operator does the job pretty well in this case.

```php
class Currency
{
// ...

    public function equals(Currency $currency)
    {
        return $currency->isoCode() === $this->isoCode();
    }
}

```



Because Money objects use Currency objects, the equals method needs to perform both this comparability check, along with comparing the amounts.

```php
class Money
{
// ...

    public function equals(Money $money)
    {
        return
            $money->currency()->equals($this->currency()) &&
            $money->amount() === $this->amount();
    }
}
```



