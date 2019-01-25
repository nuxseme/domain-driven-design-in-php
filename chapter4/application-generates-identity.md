If the client cannot provide the identity generally the preferred way to handle the identity operation is to let the application generate the identities, usually through a UUID. There are several libraries in PHP that generate UUIDs. An they can be found at packagist.

[https://packagist.org/search/?q=uuid](https://packagist.org/search/?q=uuid)

The best recommended would be the one developed by Ben Ramsey at [https://github.com/ramsey/uuid](https://github.com/ramsey/uuid)

because it has about 500 watchers on Github and about 240.000 installations on packagist, at the time of writing.

The preferred place to put the creation of the identity would be inside a Repository

```php
namespace Ddd\Billing\Domain\Model\Order;

interface OrderRepository
{
    public function nextIdentity();
    public function add(Order $anOrder);
    public function remove(Order $anOrder);
}

class OrderId
{
    private $id;

    private function construct($anId)
    {
        $this->id = $anId;
    }

    public static function create($anId)
    {
        return new static($anId);
    }
}

namespace Ddd\Billing\Infrastructure\Doctrine\Order;

use Ddd\Billing\Domain\Model\Order\Order;
use Ddd\Billing\Domain\Model\Order\OrderId;
use Ddd\Billing\Domain\Model\Order\OrderRepository;

use Doctrine\ORM\EntityRepository;
use Rhumsaa\Uuid\Uuid;

class DoctrineOrderRepository extends EntityRepository implements OrderRepository
{
    public function nextIdentity()
    {
        return OrderId::create( strtoupper(Uuid::uuid4())
        );
    }

    public function add(Order $anOrder)
    {
        $this->getEntityManager()->persist($anOrder);
    }

    public function remove(Order $anOrder)
    {
        $this->getEntityManager()->remove($anOrder);
        
    }
}
```



