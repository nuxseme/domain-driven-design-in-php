If the addition of searching capabilities to the Value Objects attributes is not important, there is another pattern that can be considered, the Serialized LOB. This pattern works by serializing the whole Value Object into a string format that can be persisted and fetched easily. The most significant difference between this solution and the Embedded alternative is that in the latter option the persistence footprint requirements get reduced to a single column.

如果向值对象属性添加搜索功能不重要，那么可以考虑另一种模式，序列化LOB。此模式的工作方式是将整个值对象序列化为字符串格式，该格式可以轻松地持久化和获取。此解决方案与嵌入式替代方案之间最显著的区别是，在后一种方案中，持久性占用需求减少到单个列

```php
CREATE TABLE `products` ( id INT NOT NULL,
name VARCHAR(255) NOT NULL,
price TEXT NOT NULL
)
```

In order to persist Product Entities using this approach, a change in the DbalProductRepository is required. The Money Value Object must be serialized into a string before persisting the final Entity.

为了使用这种方法持久化产品实体，需要更改DbalProductRepository。在持久化最后一个实体之前，必须将Money值对象序列化为字符串

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

让我们看看我们的产品现在是如何在数据库中表示的。表列price是一个文本

键入包含表示9,99美元的Money对象的序列化的列。

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

但是，由于重构代码中的类时会出现问题，所以不建议使用这种方法。您能想象在将Money类从一个名称空间移动到另一个名称空间时，数据库表示中需要做哪些更改吗?如前所述，另一个权衡是缺少查询功能。无论您是否使用Doctrine，编写查询以获得比200美元更便宜的产品几乎是不可能的，同时使用序列化策略。

查询问题只能通过使用嵌入值来解决，但是，序列化重构问题可以使用专门的库来处理序列化过程。

