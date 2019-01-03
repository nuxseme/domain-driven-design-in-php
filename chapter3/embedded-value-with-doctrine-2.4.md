So what is an acceptable solution for using embedded values with Doctrine &lt; 2.5? We need to now surrogate all the Value Objects attributes in the Product Entity, meaning to create new artificial attributes that will hold the information of the Value Object. With this in place, we can map all those new attributes using Doctrine. Let’s see what impact this has on the Product Entity.

```php
class Product
{
    protected $productId; protected $name; protected $price;

    protected $surrogateCurrencyIsoCode; protected $surrogateAmount;

    public function construct($aProductId, $aName, Money $aPrice)
    {
        $this->setProductId($aProductId);
        $this->setName($aName);
        $this->setPrice($aPrice);
    }

    private function setPrice(Money $aMoney)
    {
        $this->price = $aMoney;
        $this->surrogateAmount = $aMoney->amount();
        $this->surrogateCurrencyIsoCode = $aMoney->currency()->isoCode();
    }

    private function price()
    {
        if (null === $this->price) {
            $this->price = new Money(
                $this->surrogateAmount,
                new Currency($this->surrogateCurrency)
            );
        }

        return $this->price;
    }

// ...
}
```

As you can see, there are two new attributes. One for the amount and another for the ISO code of the currency. We have also updated the setPrice method in order to keep attribute consistency when setting it. On top of this we have updated the price getter in order to return the Money Value Object built from the new fields. Let’s see how the corresponding XML Doctrine mapping should be changed.

```php
<?xml version="1.0" encoding="utf-8"?>
<doctrine-mapping
xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">
<entity
name="Product" table="product">
<id
name="id" column="id" type="string" length="255">

<generator
strategy="NONE">
</generator>
</id>
<field
name="name" type="string" length="255"
/>
<field
name="surrogateAmount" type="integer" column="price_amount"
/>
<field
name="surrogateCurrencyIsoCode" type="string" column="price_currency"
/>
</entity>
</doctrine-mapping>
```



> Surrogate attributes
>
> These two new fields do not strictly belong to the Domain, as they do not refer to infrastructure details, but are an necessity due to the lack of embeddable support in Doctrine. There are alternatives that can push these two attributes outside the pure Domain, however, this approach is simpler, easier, and as a trade-off, acceptable. There is another use in this book of surrogate attributes, you can find it when surrogating Entity identities.





If we wish to push these two attributes outside of the Domain, this can be achieved through the use of an Abstract Factory¹³. First, we need to create a new Entity in our Infrastructure folder, DoctrineProduct that would extend from Product Entity. All surrogate fields will be placed in the new class, and methods such as price or setPrice should be reimplemented. We will map Doctrine to use the new DoctrineProduct as opposed to the Product Entity. Now, we are able to fetch Entities from the database, but, what about creating a new Product? At some point, we are required to call new Product, but because we need to deal with DoctrineProduct and we do not want our Application Services to know about infrastructure details, we will need to use Factories to create Product Entities. So, in every instance where Entity creation occurs with new, you will instead now call createProduct on ProductFactory.



There could be many additional classes required to avoid placing the surrogate attributes in the original Entity. As such, it is our recommendation to surrogate all the Value Objects to the same Entity, though this leads to a less pure solution.



