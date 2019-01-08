We have talked about the benefits of trying to model out everything in the domain as a value object first. But when modeling the domain, there will be probably situations where you will find that some concept in the ubiquitous language will be demanding a thread of identity.

Clear examples of this would be



A person. A person has always an identity and it’s always the same regarding their name, or document identifier.

An order in an e-commerce system. In that context every new order created has its own identity and it’s the same over time.



This kind of concept, have an identity that endures over the time. In PHP that would be plain old classes. For example, in the case of a person



```php
namespace Ddd\Identity\Domain\Model;

class Person
{
    private $identificationNumber;
    private $firstName;
    private $lastName;

    public function construct($anIdentificationNumber, $aFirstName, $aLastName)
    {
        $this->identificationNumber = $anIdentificationNumber;
        $this->firstName = $aFirstName;
        $this->lastName  =  $aLastName;
    }

    public function identificationNumber()
    {
        return $this->identificationNumber;
    }
    public function firstName()
    {
        return $this->firstName;
    }

    public function lastName()
    {
        return $this->lastName;
    }
}
```

Or in the case of an order, would be



```php
namespace Ddd\Billing\Domain\Model\Order;

class Order
{
    private $id; private  $amount; private $firstName; private $lastName;

    public  function      construct($anId,  Amount  $amount,  $aFirstName,  $aLastName)
    {
        $this->id = $anId;
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
}
```



