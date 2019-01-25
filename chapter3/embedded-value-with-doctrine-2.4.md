So what is an acceptable solution for using embedded values with Doctrine &lt; 2.5? We need to now surrogate all the Value Objects attributes in the Product Entity, meaning to create new artificial attributes that will hold the information of the Value Object. With this in place, we can map all those new attributes using Doctrine. Let’s see what impact this has on the Product Entity.

那么在Doctrine &lt; 2.5中使用嵌入值的可接受解决方案是什么呢?现在我们需要代理产品实体中的所有值对象属性，这意味着创建新的人工属性来保存值对象的信息。有了这些，我们就可以使用Doctrine映射所有这些新属性。让我们看看这对产品实体有什么影响

```php
class Product
{
    protected $productId; 
    protected $name; 
    protected $price;

    protected $surrogateCurrencyIsoCode; 
    protected $surrogateAmount;

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

如您所见，有两个新属性。一个用于金额，另一个用于货币的ISO代码。我们还更新了setPrice方法，以便在设置它时保持属性的一致性。在此之上，我们更新了price getter，以返回从新字段构建的Money Value对象。让我们看看应该如何更改相应的XML Doctrine映射

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
>
> 代理属性
>
> 这两个新字段并不严格属于域，因为它们不涉及基础设施细节，而是由于学说中缺乏可嵌入的支持而必须的。有一些替代方法可以将这两个属性推到纯域之外，但是，这种方法更简单、更容易，作为一种权衡，也是可以接受的。在这本关于代理属性的书中还有另一个用途，您可以在代理实体身份时找到它。

If we wish to push these two attributes outside of the Domain, this can be achieved through the use of an Abstract Factory¹³. First, we need to create a new Entity in our Infrastructure folder, DoctrineProduct that would extend from Product Entity. All surrogate fields will be placed in the new class, and methods such as price or setPrice should be reimplemented. We will map Doctrine to use the new DoctrineProduct as opposed to the Product Entity. Now, we are able to fetch Entities from the database, but, what about creating a new Product? At some point, we are required to call new Product, but because we need to deal with DoctrineProduct and we do not want our Application Services to know about infrastructure details, we will need to use Factories to create Product Entities. So, in every instance where Entity creation occurs with new, you will instead now call createProduct on ProductFactory.

There could be many additional classes required to avoid placing the surrogate attributes in the original Entity. As such, it is our recommendation to surrogate all the Value Objects to the same Entity, though this leads to a less pure solution.

如果我们希望推动这两个属性以外的领域,这可以通过使用一个抽象工厂¹³。首先，我们需要在基础结构文件夹中创建一个新实体，DoctrineProduct，它将从Product Entity扩展而来。所有代理字段都将放置在新类中，price或setPrice等方法应该重新实现。我们将把Doctrine映射为使用新的DoctrineProduct而不是Product实体。现在，我们能够从数据库中获取实体，但是，创建一个新产品如何呢?在某种程度上，我们需要调用新产品，但是因为我们需要处理DoctrineProduct，并且我们不希望应用程序服务知道基础结构细节，所以我们需要使用工厂来创建产品实体。因此，在使用new创建实体的每个实例中，您现在将在ProductFactory上调用createProduct。

可能需要许多额外的类来避免将代理属性放在原始实体中。因此，我们建议将所有值对象替换为相同的实体，尽管这会导致不太纯的解决方案。

