Currency and Money Value Objects are probably the most used examples for explaining Value Objects thanks to the Money pattern⁴. This design pattern provides a solution to model the problem in order to avoid floating-point rounding issue, allowing for deterministic calculations to be performed.

In the real world a currency describes monetary units in the same way as meters and yards describe distance units. Each currency is represented with a three upper-case letter ISO code.



由于货币模式，货币和货币价值对象可能是解释价值对象最常用的例子。此设计模式提供了对问题建模的解决方案，以避免浮点四舍五入问题，从而允许执行确定性计算。

在现实世界中，货币描述货币单位就像米和码描述距离单位一样。每种货币都用三个大写字母ISO代码表示。

```php
class Currency
{
    private $isoCode;

    public function construct($anIsoCode)
    {
        $this->setIsoCode($anIsoCode);
    }

    private function setIsoCode($anIsoCode)
    {
        if (!preg_match('/^[A-Z]{3}$/', $anIsoCode)) {
            throw new \InvalidArgumentException();
        }

        $this->isoCode = $anIsoCode;
    }

    public function isoCode()
    {
        return $this->isoCode;
    }
}
```

One of the main goals of Value Objects is also the holy grail of Object Oriented design: encapsulation. By following this abstraction, you will end up with a dedicated location to put all the validation, comparison logic and behaviour for a given concept.

Money is used to measure a specific amount of currency. It is modelled using an amount and a Currency. Amount, in the case of the Money pattern, is implemented using an integer representation of the currency’s least-valuable fraction - i.e. in the case of USD or EUR, cents.

As a bonus point, you will also notice in the example that we are using self-encapsulation⁵ to set the ISO code, centralising changes in the Value Object itself.



Value对象的主要目标之一也是面向对象设计的圣杯:封装。通过遵循这个抽象，您最终将得到一个专门的位置来存放给定概念的所有验证、比较逻辑和行为。

货币是用来衡量一定数量的货币。它是使用数量和货币进行建模的。在货币模式中，Amount是使用货币的最小值部分的整数表示形式实现的，即在美元或欧元的情况下，使用美分表示。



```php
class Money
{
    private  $amount;
    private $currency;

    public  function  construct($anAmount,  Currency  $aCurrency)
    {
        $this->setAmount($anAmount);
        $this->setCurrency($aCurrency);
    }

    private function setAmount($anAmount)
    {
        $this->amount  =  (int)  $anAmount;
    }

    private function setCurrency(Currency $aCurrency)
    {
        $this->currency = $aCurrency;
    }

    public function amount()
    {
        return  $this->amount;
    }

    public function currency()
    {
        return $this->currency;
    }
}
```

Now that you know the formal definition of a Value Object, let’s dive deeper into some of the powerful features that they offer.



现在您已经了解了值对象的正式定义，接下来让我们深入了解它们提供的一些功能强大的特性

---

...

