If we are dealing with an ad-hoc ORM using the Embedded Value pattern, we need to create a field in the entity table for each attribute in the Value Object. In this case, two extra columns are needed when persisting a Product Entity, one for the amount of the Value Object, and the second for its currency ISO code.

如果我们使用嵌入式值模式处理一个特殊的ORM，我们需要在实体表中为值对象中的每个属性创建一个字段。在这种情况下，在持久化产品实体时需要另外两列，一列表示值对象的数量，另一列表示其货币ISO代码

```php
CREATE TABLE `products` ( id  INT  NOT  NULL,
name  VARCHAR(255)  NOT  NULL,
price_amount  INT  NOT  NULL, price_currency  VARCHAR(3)  NOT  NULL
)
```

For persisting the Entity in the database, our Repository has to map one-to-one each of the fields of the Entity and the ones from the Money Value Object. If using an ad-hoc ORM repository based on DBAL, the DbalProductRepository should create the INSERT statement, bind the parameters and execute it.

为了持久化数据库中的实体，我们的存储库必须一对一地映射实体的每个字段和Money Value对象的字段。如果使用基于DBAL的特殊ORM存储库，DbalProductRepository应该创建INSERT语句，绑定参数并执行它

```php
class DbalProductRepository extends DbalRepository implements ProductRepository
{
    public function add(Product $aProduct)
    {
        $sql = 'INSERT INTO products VALUES (?, ?, ?, ?)';
        $stmt = $this->connection()->prepare($sql);
        $stmt->bindValue(1,  $aProduct->id());
        $stmt->bindValue(2,  $aProduct->name());
        $stmt->bindValue(3,  $aProduct->price()->amount());
        $stmt->bindValue(4,  $aProduct->price()->currency()->isoCode());
        $stmt->execute();

// ...
    }
}
```

After executing this snippet of code to create a Product Entity and persist it into the database, each column has been filled with the desired information.

在执行这段代码片段以创建产品实体并将其持久化到数据库中之后，每一列都填充了所需的信息

```php
mysql> select * from products \G
***************************  1.  row  ***************************  id: 1
name: Domain-Driven Design in PHP price_amount: 999
price_currency:   USD
1 row in set (0.00 sec)
```

As you can see, you can map your Value Objects and query parameters in an ad-hoc manner to persist your Value Objects. However, everything is not as easy as it seems. Let’s try to fetch the persisted Product with its associated Money Value Object. A common approach would be to execute a SELECT statement and return a new Entity.

如您所见，您可以以一种特殊的方式映射值对象和查询参数，以持久化值对象。然而，事情并不像看上去那么简单。让我们尝试获取具有关联Money Value对象的持久化产品。一种常见的方法是执行SELECT语句并返回一个新实体

```php
class DbalProductRepository extends DbalRepository implements ProductRepository
{
    public function productOfId($anId)
    {
        $sql = 'SELECT * FROM products WHERE id = ?';
        $stmt = $this->connection()->prepare($sql);
        $stmt->bindValue(1,  $anId);
        $res = $stmt->execute();
// ...

        return new Product(
            $row['id'],
            $row['name'],
            new Money(
                $row['price_amount'],
                new Currency(
                    $row['price_currency']
                )
            )
        );
    }
}
```

There are some benefits to this approach. First is that you can easily read step-by-step how the persistence and subsequent creation is occurring. Second, you can perform queries based on any of the attributes of the Value Object. Finally, the space required to persist the Entity is just what is required, no more, no less.

However, using the ad-hoc ORM approach has its drawbacks. As explained in the Domain Events chapter, Entities \(in Aggregate form\) should fire an Event in the constructor if your Domain is interested in the Aggregates creation. If you use the new operator, you would be firing the event as many times as the Aggregate is fetched from the database.

That is one of the reasons why Doctrine uses internally Proxies, serialize, and unserialize methods to reconstitute an object with its attributes in a specific state without using its constructor. An Entity should be created with the new operator just once in its lifetime.

这种方法有一些好处。首先，您可以轻松地一步一步地阅读持久性和后续创建是如何发生的。其次，您可以基于值对象的任何属性执行查询。最后，持久化实体所需的空间正是所需的，不多也不少。

然而，使用特别ORM方法有其缺点。正如域事件一章中所解释的，如果域对聚合创建感兴趣，实体\(聚合形式\)应该在构造函数中触发事件。如果使用新操作符，则触发事件的次数将与从数据库获取聚合的次数相同。

这就是Doctrine使用内部代理、序列化和非序列化方法在不使用其构造函数的情况下用其属性在特定状态下重新构造对象的原因之一。在实体的生命周期中，应该只使用新操作符创建一次实体。



> Constructors
>
> Constructors do not need to include a parameter for each attribute in the object. Think about a blog Post. A constructor may need an id and a title, however, internally it can also be setting its status attribute to draft. When publishing the post, a publish method should be called in order to alter its status accordingly and set a published date.
>
> 构造函数不需要为对象中的每个属性包含参数。想想一篇博客文章。构造函数可能需要id和标题，但是，在内部它也可以将其状态属性设置为draft。发布文章时，应该调用发布方法来相应地更改其状态并设置发布日期

If your intention is still on rolling out your own ORM, be ready to solve some fundamental problems, such as events, different constructors, Value Objects, lazy load relations, etc. That is why we recommend giving Doctrine a try for DDD applications.

Besides, in this instance, you need to create a DbalProduct Entity that extends from the Product Entity and is able to reconstitute the Entity from the database without using the new operator, using a static factory method.

如果您的意图仍然是推出您自己的ORM，那么请准备好解决一些基本问题，例如事件、不同的构造函数、值对象、延迟加载关系等等。这就是为什么我们建议为DDD应用程序尝试Doctrine的原因。

此外，在本例中，您需要创建一个DbalProduct实体，该实体从产品实体扩展而来，并且能够在不使用新操作符的情况下使用静态工厂方法从数据库中重新构建该实体

