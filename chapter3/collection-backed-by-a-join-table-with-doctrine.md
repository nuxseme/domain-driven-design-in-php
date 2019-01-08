Doctrine requires that all database entities to have a unique identity. Because we want to persist Money Value Objects we need to then add an artificial identity so Doctrine can handle its persistence. There are two options: including the surrogate identity in the Money Value Object or placing it in an extended class.

The issue with the first option is that the new identity is only required due to the Database persistence layer. This identity is not part of the Domain.

An issue with the second option is the amount of alteration that are required in order to avoid this said boundary leak. With a class extension, creating new instances of the Money Value Object class from any Domain Object is not recommended, as it would break the Inversion Principle. The solution is to again create a Money Factory that would need to be passed into Application Services and any other Domain objects.

In this scenario, we recommend to use the first option. Let’s review the changes required to the

Money Value Object.



```php
class Money
{
    private $amount; private $currency;

    private $surrogateId;
    private $surrogateCurrencyIsoCode;

    public function construct($amount, Currency $currency)
    {
        $this->setAmount($amount);
        $this->setCurrency($currency);
    }

    private function setAmount($amount)
    {
        $this->amount = $amount;
    }

    private function setCurrency(Currency $currency)
    {
        $this->currency = $currency;
        $this->surrogateCurrencyIsoCode = $currency->isoCode();
    }

    public function currency()
    {
        if (null === $this->currency) {
            $this->currency = new Currency($this->surrogateCurrencyIsoCode);
        }

        return $this->currency;
    }

    public function amount()
    {
        return $this->amount;
    }

    public function equals(Money $aMoney)
    {
        return $this->amount() === $aMoney->amount() && $this->currency()->equals($this->currency());
    }
}
```



As seen, two new attributes have been added. The first one, surrogateId is not used by our Domain, but is required for the persistence infrastructure to persist this Value Object as an Entity in our Database. The second one, surrogateCurrencyIsoCode holds the ISO code for the currency. Using these new attributes it is really easy to map our Value Object with Doctrine.

The Money mapping is quite straight forward.



```php
<?xml version="1.0" encoding="utf-8"?>
<doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mappin\ g" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="htt\ p://doctrine-project.org/schemas/orm/doctrine-mapping https://raw.github.com/doc\ trine/doctrine2/master/doctrine-mapping.xsd">
<entity name="Ddd\Domain\Model\Money" table="prices">
<id name="surrogateId" type="integer" column="id">
<generator strategy="AUTO"></generator>
</id>
<field name="amount" type="integer" column="amount"/>
<field name="surrogateCurrencyIsoCode" type="string" column="currency"/>
</entity>
</doctrine-mapping>
```

Using Doctrine, the HistoricalProduct Entity would have following mapping.



```php
<?xml version="1.0" encoding="utf-8"?>
<doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mappin\ g" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="htt\ p://doctrine-project.org/schemas/orm/doctrine-mapping https://raw.github.com/doc\ trine/doctrine2/master/doctrine-mapping.xsd">
<entity name="Ddd\Domain\Model\HistoricalProduct" table="historical_products" \ repository-class="Ddd\Infrastructure\Domain\Model\DoctrineHistoricalProductRepos\ itory">
<many-to-many field="prices" target-entity="Ddd\Domain\Model\Money">
<cascade>
<cascade-all/>
</cascade>
<join-table name="products_prices">
<join-columns>

<join-column name="product_id" referenced-column-name="id" />
</join-columns>
<inverse-join-columns>
<join-column name="price_id" referenced-column-name="id" unique="true"\
/>
</inverse-join-columns>
</join-table>
</many-to-many>
</entity>
</doctrine-mapping>
```



