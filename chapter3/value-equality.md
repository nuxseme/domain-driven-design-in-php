As discussed at the beginning of the chapter, two Value Objects are equal if the content they measure, quantify, or describe is the same.

For example, conceptualise two Money objects representing 1 USD. Can we consider them equal? In the ‘real’ world are two coins of 1 USD valued the same? Of course they are. Directing our attention back to the code, the Value Objects in question refers to separate instances of Money. However, we can consider them to both represent the same value, so in-turn they are equal.

In regard to PHP, it is common place to compare two Value Objects using the == operator. Examining the PHP documentations⁹ definition of the operator highlights an interesting behaviour.

When using the comparison operator \(==\), object variables are compared in a simple manner, namely: Two object instances are equal if they have the same attributes and values, and are instances of the same class.

This behaviour works in agreement to our formal definition of a Value Object. However, as an exact class match predicate is present, you should be wary when handling sub-typed Value Objects.

With this in mind, the even stricter === operator unfortunately does not help us.

When using the identity operator \(===\), object variables are identical if and only if they refer to the same instance of the same class.

The following example should help confirm these subtle differences.



如本章开头所讨论的，如果度量、量化或描述的内容相同，则两个值对象是相等的。

例如，概念化两个表示1美元的货币对象。我们可以认为它们相等吗?在“现实”世界中，两枚1美元的硬币价值相同吗?他们当然是。将我们的注意力引回到代码上，所讨论的值对象指的是货币的不同实例。然而，我们可以认为它们都表示相同的值，因此它们是相等的。

对于PHP，通常使用==运算符比较两个值对象。检查PHP文档⁹操作符的定义强调了一个有趣的行为。

当使用比较运算符\(==\)时，对象变量以一种简单的方式进行比较，即:如果两个对象实例具有相同的属性和值，并且是相同类的实例，那么它们是相等的。

这种行为符合我们对值对象的正式定义。但是，由于存在精确的类匹配谓词，所以在处理子类型值对象时应该小心。

考虑到这一点，更严格的===运算符对我们没有帮助。

当使用恒等运算符\(===\)时，当且仅当对象变量引用相同类的相同实例时，它们是相同的。

下面的示例应该有助于确认这些细微的差异。

```php
$a   =   new Currency('USD');
$b   =   new Currency('USD');

var_dump($a == $b); // bool(true)
var_dump($a === $b); // bool(false)
$c = new Currency('EUR'); var_dump($a == $c);    //  bool(false)
var_dump($a === $c); // bool(false)
```

With this in mind a solution is to implement a conventional equals method in each Value Object. This method is tasked with checking the type and equality of its composite attributes. Abstract data type comparability is easy to implement using PHP’s built-in type hinting. On the other hand you can also use the get\_class\(\) function to aid in the comparability check if necessary. The language however, is unable to decipher what equality truly means in your domain concept, meaning it is your responsibility to provide the answer.

In order to compare Currency objects, we just need to compare both their associated ISO codes are the same. The === operator does the job pretty well in this case.

记住这一点，解决方案是在每个值对象中实现常规的equals方法。此方法的任务是检查其复合属性的类型和相等性。使用PHP内置的类型提示很容易实现抽象数据类型的可比性。另一方面，如果需要，还可以使用get\_class\(\)函数来帮助进行可比性检查。然而，该语言无法解释在您的领域概念中平等的真正含义，这意味着您有责任提供答案。

为了比较货币对象，我们只需要比较它们相关的ISO代码是否相同。在这种情况下，===运算符可以很好地完成这项工作。



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

因为Money对象使用Currency对象，equals方法需要同时执行这种可比性检查和金额比较。



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



