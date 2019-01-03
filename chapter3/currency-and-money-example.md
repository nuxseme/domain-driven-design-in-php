Currency and Money Value Objects are probably the most used examples for explaining Value Objects thanks to the Money pattern⁴. This design pattern provides a solution to model the problem in order to avoid floating-point rounding issue, allowing for deterministic calculations to be performed.

In the real world a currency describes monetary units in the same way as meters and yards describe distance units. Each currency is represented with a three upper-case letter ISO code.

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



```php
class Money
{
    private  $amount;
    private $currency;

    public  function      construct($anAmount,  Currency  $aCurrency)
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

