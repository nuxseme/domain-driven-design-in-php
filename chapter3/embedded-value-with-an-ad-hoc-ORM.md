If we are dealing with an ad-hoc ORM using the Embedded Value pattern, we need to create a field in the entity table for each attribute in the Value Object. In this case, two extra columns are needed when persisting a Product Entity, one for the amount of the Value Object, and the second for its currency ISO code.

```php
CREATE TABLE `products` ( id  INT  NOT  NULL,
name  VARCHAR(255)  NOT  NULL,
price_amount  INT  NOT  NULL, price_currency  VARCHAR(3)  NOT  NULL
)
```

For persisting the Entity in the database, our Repository has to map one-to-one each of the fields of the Entity and the ones from the Money Value Object. If using an ad-hoc ORM repository based on DBAL, the DbalProductRepository should create the INSERT statement, bind the parameters and execute it.

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

```php
mysql> select * from products \G
***************************  1.  row  ***************************  id: 1
name: Domain-Driven Design in PHP price_amount: 999
price_currency:   USD
1 row in set (0.00 sec)

```

As you can see, you can map your Value Objects and query parameters in an ad-hoc manner to persist your Value Objects. However, everything is not as easy as it seems. Let’s try to fetch the persisted Product with its associated Money Value Object. A common approach would be to execute a SELECT statement and return a new Entity.

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



> Constructors
>
> Constructors do not need to include a parameter for each attribute in the object. Think about a blog Post. A constructor may need an id and a title, however, internally it can also be setting its status attribute to draft. When publishing the post, a publish method should be called in order to alter its status accordingly and set a published date.



If your intention is still on rolling out your own ORM, be ready to solve some fundamental problems, such as events, different constructors, Value Objects, lazy load relations, etc. That is why we recommend giving Doctrine a try for DDD applications.

Besides, in this instance, you need to create a DbalProduct Entity that extends from the Product Entity and is able to reconstitute the Entity from the database without using the new operator, using a static factory method.





