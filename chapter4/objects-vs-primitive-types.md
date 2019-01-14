Most of the time the identity of an entity is represented as a primitive type: usually a string or an integer. But using a value object to represent it has more advantages:



Value Objects are immutable, so they cannot be modified.

Value Objects are complex types that can have custom behaviours that otherwise with primitive types cannot have. Put for example the equality operation. With value objects, equality operations can be modelled and encapsulated in their own classes, making concepts go from implicit to explicit.

```php
namespace Ddd\Billing\Domain\Model;

class OrderId
{
    private $id;

    public function construct($anId)
    {
        $this->id = $anId;
    }

    public function id()
    {
        return $this->id;
    }

    public function equalsTo(OrderId $anOrderId)
    {
        return $anOrderId->id === $this->id;
    }
}

class Order
{
    private $id;
    private $amount;
    private $firstName; 
    private $lastName;

    public  function construct(OrderId  $anOrderId,  Amount  $amount,  $aFirstName,\$aLastName)
    {
        $this->id = $anOrderId;
        $this->amount  =  $amount;
        $this->firstName = $aFirstName;
        $this->lastName  =  $aLastName;
    }

    public function id()
    {
        return $this->id;
    }

    public function firstName()
    {
        return $this->firstName;
    }
    
    public function lastName()
    {
        return $this->lastName;
    }
    
    public function amount()
    {
        return  $this->amount;
    }
}
```



