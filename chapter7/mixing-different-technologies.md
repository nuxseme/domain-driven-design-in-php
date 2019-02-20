In large business-critical applications it’s quite common to have a mix of several technologies. For

example, in read-intensive web applications you usually have some sort of denormalized data source

\(Solr, Elastic, Sphinx, etc.\) that provides all the reads of the application while a traditional RDBMS

like MySQL or Postgres is mainly responsible to handle all the writes. When this occurs one of the

concerns that normally arise is whether we can have read operations go with the search engine and

the write operations go with the traditional RDBMS data source. Our general advice here, is that

these kind of situations are a smell for CQRS, since we are in the need to scale the reads and the

writes of the application independently. So if you can go with CQRS, probably that will be the best

choice.

在大型业务关键型应用程序中，混合使用几种技术是很常见的。为例如，在读取密集型web应用程序中，通常有某种非规范化数据源\(Solr、Elastic、Sphinx等\)，它在传统RDBMS中提供应用程序的所有读操作像MySQL或Postgres主要负责处理所有的写。当这种情况发生时通常出现的问题是，我们是否可以让读取操作与搜索引擎和写操作使用传统的RDBMS数据源。我们的建议是这种情况对于CQRS来说是一种味道，因为我们需要调整读取和独立编写应用程序。所以如果你可以使用CQRS，那可能是最好的选择。

But if for any reason you cannot go with CQRS, an alternative approach is needed. In this situation,

the use of the Proxy pattern from Gang of Four comes in handy. We can define an implementation

of a repository in terms of the Proxy pattern.

但是，如果由于某种原因您不能使用CQRS，则需要另一种方法。在这种情况下,使用“四人帮”的代理模式很方便。我们可以定义一个实现存储库的代理模式。

```php
namespace BuyIt\Billing\Infrastructure\FullTextSearching\Elastica;
use BuyIt\Billing\DomainModel\Order\OrderRepository;
use BuyIt\Billing\Infrastructure\Persistence\Doctrine\DoctrineOrderRepository;
use Elastica\Client;
class ElasticaOrderRepository implements OrderRepository
{
    private $client;
    private $baseOrderRepository;
    public function __construct(Client $client, DoctrineOrderRepository $baseOrderRepository)
    {
        $this->client = $client;
        $this->baseOrderRepository = $baseOrderRepository;
    }
    public function find($id)
    {
        return $this->baseOrderRepository->find($id);
    }
    public function findBy(array $criteria)
    {
        $search = new \Elastica\Search($this->client);
// ...
        return $this->toOrder($search->search());
    }
    public function add($anOrder)
    {
// First we attempt to add it to the Elastic index
        $ordersIndex = $this->client->getIndex('orders');
        $orderType = $ordersIndex->getType('order');
        $orderType->addDocument(
            new \Elastica\Document(
                $anOrder->id(),
                $this->toArray($anOrder)
            )
        );
        $ordersIndex->refresh();
// When it's done, we attempt to add it to the RDBMS store
        $this->baseOrderRepository->add($anOrder);
    }
}
```

This example provides a naive implementation using the DoctrineOrderRepository and the Elastica

client, a client to interact with an Elastic server. Note that for some operations we are using the

RDBMS datasource and for others the Elastica client. And also note that the add operation consists

of two parts. The first one attempts to store the Order to the Elastic index and the second one

attempts to store the Order into the relational database delegating the operation to the Doctrine

implementation. Take into account that this is just an example and a way to do it. Probably it can

be improved, for example now the whole add operation is synchronous. We could instead enqueue

the operation to some sort of messaging middleware that stores the Order into Elastic, for example.

There are a lot of posibilities and improvements, depending on your needs.

这个例子提供了一个使用DoctrineOrderRepository和Elastica的简单实现客户端，与弹性服务器交互的客户端。注意，对于某些操作，我们使用RDBMS数据源和其他的Elastica客户端。还要注意，add操作包括两部分组成。第一个尝试将顺序存储到弹性指标，第二个尝试存储到弹性指标尝试将订单存储到将操作委托给Doctrine的关系数据库中实现。考虑到这只是一个例子和一种方法。也许它可以改进，例如现在整个add操作是同步的。我们可以排队例如，将订单存储为弹性的某种消息传递中间件的操作。根据您的需要，有许多可能性和改进。

