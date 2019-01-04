Value Objects are not persisted on their own, they are typically persisted within an Aggregate. Value Objects should not be persisted as complete records, though it is an option in some cases. Instead it is best to use Embedded Value or Serialize LOB patterns. Both patterns can be used when persisting your objects with an open-source ORM such as Doctrine or with a bespoke ORM. As Value Objects are small, Embedded Value is usually the best choice because it allows an easy way to query Entities by any of the attributes the Value Object has. However, if querying by those fields is not important to you, Serialize strategies can be very easy to implement.

Consider the following Product Entity with a string id, name, and price \(Money Value Object\) attributes. We have intentionally decided to simplify this example with the id being a string and not a Value Object.



值对象本身并不持久，它们通常在聚合中持久。值对象不应该作为完整的记录保存，尽管在某些情况下它是一个选项。相反，最好使用嵌入式值或序列化LOB模式。当使用开源ORM\(如Doctrine\)或定制ORM来持久化对象时，可以使用这两种模式。由于值对象很小，嵌入式值通常是最佳选择，因为它允许通过值对象的任何属性轻松查询实体。然而，如果按这些字段进行查询对您来说并不重要，那么序列化策略可以非常容易地实现。

考虑以下具有字符串id、名称和price \(Money Value Object\)属性的产品实体。我们有意将这个示例简化为一个字符串，而不是一个值对象。

```php
class Product
{
    private $productId;
    private  $name; 
    private $price;

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

Assuming you have a Repository for persisting Product Entities, an implementation to create and persist a new Product could look like the following.

Let’s now look at both the ad-hoc ORM and the Doctrine implementations which could be used to persist a Product Entity which contains Value Objects. We will highlight the application of the Embedded Value and Serialized LOB patterns, and the differences between persisting a single Value Object and a collection of them.

假设您有一个用于持久化产品实体的存储库，那么创建和持久化新产品的实现可能如下所示。

现在让我们来看看特殊ORM和Doctrine实现，它们可以用于持久化包含值对象的产品实体。我们将重点介绍嵌入式值和序列化LOB模式的应用程序，以及持久化单个值对象和它们的集合之间的区别。



```php
$product = new Product(
    $productRepository->nextIdentity(), 'Domain-Driven  Design  in  PHP',
    new  Money(999,  new  Currency('USD'))
);

$productRepository->persist($product);
```

> Why Doctrine?
>
> Doctrine¹⁰ is a great ORM. It solves 80% of the requirements a PHP application faces. It has a great community. With a correctly-tuned set-up, it can perform the same or even better than a bespoke ORM \(without losing maintainability\). We recommend using Doctrine in most cases when dealing with Entities and business logic. It will save you a lot of time and headaches.
>
> Doctrine是一个伟大的ORM。它解决了PHP应用程序所面临的80%的需求。它有一个很棒的社区。通过正确调优的设置，它可以执行与定制ORM相同甚至更好的操作\(而不会丧失可维护性\)。我们建议在大多数情况下在处理实体和业务逻辑时使用Doctrine。它会节省你很多时间和头痛。

---



