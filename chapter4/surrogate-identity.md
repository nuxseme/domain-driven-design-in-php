Sometimes using an ORM to map entities to a persistence store, some constraints are imposed, for example Doctrine demands of an integer field if an IDENTITY generator strategy is used. This can conflict with the domain model if it requires another kind of identity.

The simplest way to handle that situation is by using a Layer SuperType¹ where we put the identity field created for the persistence store.



```php
namespace Ddd\Common\Domain\Model;

abstract class IdentifiableDomainObject
{
    private $id;

    protected function id()
    {
        return $this->id;
    }

    protected function setId($anId)
    {
        $this->id = $anId;
    }
}

namespace Acme\Billing\Domain;

use Acme\Common\Domain\IdentifiableDomainObject;

class Order extends IdentifiableDomainObject
{
    private $orderId;

    public function orderId()
    {
        if (null === $this->orderId) {
            $this->orderId = new OrderId($this->id());
        }

        return $this->orderId;
    }
}
```



