If the addition of searching capabilities to the Value Objects attributes is not important, there is another pattern that can be considered, the Serialized LOB. This pattern works by serializing the whole Value Object into a string format that can be persisted and fetched easily. The most significant difference between this solution and the Embedded alternative is that in the latter option the persistence footprint requirements get reduced to a single column.

```php
CREATE TABLE `products` ( id INT NOT NULL,
name VARCHAR(255) NOT NULL,
price TEXT NOT NULL
)
```

In order to persist Product Entities using this approach, a change in the DbalProductRepository is required. The Money Value Object must be serialized into a string before persisting the final Entity.

```php
class DbalProductRepository extends DbalRepository implements ProductRepository
{
    public function add(Product $aProduct)
    {
        $sql  = 'INSERT INTO products VALUES (?, ?, ?)';
        $stmt = $this->connection()->prepare($sql);
        $stmt->bindValue(1, $aProduct->id());
        $stmt->bindValue(2, $aProduct->name());
        $stmt->bindValue(3,
            $this->serialize(
                $aProduct->price()
            )
        );

// ...
    }

    private function serialize($object)
    {
        return serialize($object);

    }
}
```

Let’s see how our Product is now represented in the database. The table column price is a TEXT

type column that contains a serialization of a Money object representing 9,99 USD.



```php
mysql>  select  *  from  products  \G
***************************  1.  row  ***************************
id: 1
name:  Domain-Driven  Design  in  PHP
price:  O:22:"Ddd\Domain\Model\Money":2:{s:30:"  Ddd\Domain\Model\Money  amount";i:\ 999;s:32:"  Ddd\Domain\Model\Money  currency";O:25:"Ddd\Domain\Model\Currency":1:{\ s:34:"  Ddd\Domain\Model\Currency  isoCode";s:3:"USD";}}
1 row in set (0.00 sec)
```

This approach does the job, however, it is not recommended due to problems occurring when refactoring classes in your code. Could you imagine the changes that would be required in our database representation, when moving the Money class from one namespace to another? Another trade-off, as explained before, is the lack of querying capabilities. It does not matter whether you use Doctrine or not, writing a query to get the products cheaper than say 200 USD is almost impossible whilst using a serialization strategy.

The querying issue can only be solved by using Embedded Values, however, the serialization refactoring problems can be fixed using a specialised library for handling serialization processes.



