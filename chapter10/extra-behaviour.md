Adding additional behaviour to a repository can be very beneficial, such as the ability to count all the items in a given collection. You might think to add a method with the name count however, as we are trying to mimic a collection, a better name would instead be size.

向存储库中添加额外的行为可能非常有益，例如能够计算给定集合中的所有项。您可能会认为添加名称计数的方法，但是，当我们试图模拟集合时，更好的名称应该是size。

```php
interface PostRepository
{

    public function size();
}
```

And the implementation could look like

它的实现是这样的

```php
class DoctrinePostRepository implements PostRepository
{

    public function size()
    {
        return  $this->em->createQueryBuilder()
            ->select('count(p.id)')
            ->from('Domain\Model\Post', 'p')
            ->getQuery()
            ->getSingleScalarResult();
    }
}
```

You are able to also encapsulate calculations into the repository, along with data-storage specific and optimised queries/stored procedures. All behaviour should still however, follow the repositories collection characteristics. It is encouraged to move as much logic into domain-specific stateless Domain Services as possible, instead of simply adding these responsibilities to the repository.

In some instances you will not require the entire Aggregate for simply accessing small amounts of information. To solve this you can add repository methods to access these as shortcuts. You should make sure to only access data that could be retrieved by navigating through the Aggregate Root. As such you should not allow access to the Aggregate Roots private and internal areas, as this would violate the laid out contractual agreement.

For some use cases you will require very specific queries that are compositions of multiple Aggregate types, each returning specific information. These queries can be run and then returned as a single Value Object. It is very common for repositories to return Value Objects.

If you find yourself creating many use-case optimal finder methods, you may be introducing a common code smell. This could be an indication of a misjudged Aggregate boundary. If however, you are confident that the boundaries are correct, it could be time to explore CQRS.



您还可以将计算以及特定于数据存储的优化查询/存储过程封装到存储库中。然而，所有的行为都应该遵循储存库集合的特征。我们鼓励将尽可能多的逻辑转移到特定于域的无状态域服务中，而不是简单地将这些职责添加到存储库中。

在某些情况下，您不需要整个聚合来简单地访问少量信息。要解决这个问题，您可以添加存储库方法来作为快捷方式访问它们。您应该确保只访问通过聚合根导航可以检索的数据。因此，您不应该允许访问私有和内部区域的聚合根，因为这将违反所制定的契约协议。

对于某些用例，您将需要非常特定的查询，这些查询是多个聚合类型的组合，每个聚合类型返回特定的信息。这些查询可以运行，然后作为单个值对象返回。存储库返回值对象是非常常见的。

如果您发现自己创建了许多用例优化查找器方法，那么您可能正在引入一种常见的代码味道。这可能表明总体边界判断错误。但是，如果您确信边界是正确的，那么就应该研究CQRS。

