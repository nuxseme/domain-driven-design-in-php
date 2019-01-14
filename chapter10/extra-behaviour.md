Adding additional behaviour to a repository can be very beneficial, such as the ability to count all the items in a given collection. You might think to add a method with the name count however, as we are trying to mimic a collection, a better name would instead be size.



```php
interface PostRepository
{

    public function size();
}

```



And the implementation could look like



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

