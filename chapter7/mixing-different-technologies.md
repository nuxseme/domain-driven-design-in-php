In large business-critical applications it’s quite common to have a mix of several technologies. For

example, in read-intensive web applications you usually have some sort of denormalized data source

\(Solr, Elastic, Sphinx, etc.\) that provides all the reads of the application while a traditional RDBMS

like MySQL or Postgres is mainly responsible to handle all the writes. When this occurs one of the

concerns that normally arise is whether we can have read operations go with the search engine and

the write operations go with the traditional RDBMS data source. Our general advice here, is that

these kind of situations are a smell for CQRS, since we are in the need to scale the reads and the

writes of the application independently. So if you can go with CQRS, probably that will be the best

choice.

But if for any reason you cannot go with CQRS, an alternative approach is needed. In this situation,

the use of the Proxy pattern from Gang of Four comes in handy. We can define an implementation

of a repository in terms of the Proxy pattern.

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

