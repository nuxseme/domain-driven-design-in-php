Value Objects are not persisted on their own, they are typically persisted within an Aggregate. Value Objects should not be persisted as complete records, though it is an option in some cases. Instead it is best to use Embedded Value or Serialize LOB patterns. Both patterns can be used when persisting your objects with an open-source ORM such as Doctrine or with a bespoke ORM. As Value Objects are small, Embedded Value is usually the best choice because it allows an easy way to query Entities by any of the attributes the Value Object has. However, if querying by those fields is not important to you, Serialize strategies can be very easy to implement.

Consider the following Product Entity with a string id, name, and price \(Money Value Object\) attributes. We have intentionally decided to simplify this example with the id being a string and not a Value Object.

```php
class Product
{
    private $productId; private  $name; private $price;

    public function construct(
        $aProductId,
        $aName,
        Money $aPrice
    ) {
        $this->setProductId($aProductId);
        $this->setName($aName);
        $this->setPrice($aPrice);
    }

// ...
}
```



