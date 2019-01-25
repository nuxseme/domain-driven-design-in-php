We have talked about the benefits of trying to model out everything in the domain as a value object first. But when modeling the domain, there will be probably situations where you will find that some concept in the ubiquitous language will be demanding a thread of identity.

Clear examples of this would be

A person. A person has always an identity and it’s always the same regarding their name, or document identifier.

An order in an e-commerce system. In that context every new order created has its own identity and it’s the same over time.

This kind of concept, have an identity that endures over the time. In PHP that would be plain old classes. For example, in the case of a person

我们已经讨论了首先将域中的所有东西建模为值对象的好处。但是，在建模领域时，您可能会发现，在普遍存在的语言中，有些概念需要标识线程。

这方面的明显例子是

一个人。一个人总是有一个身份，关于他的名字或者文档标识符总是一样的。

电子商务系统中的订单。在这种情况下，每个新创建的订单都有自己的标识，而且随着时间的推移，它是相同的。

这种观念，具有一种经久不衰的特性。在PHP中，这是普通的旧类。例如，在一个人的情况下

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
    private $id; 
    private $amount; 
    private $firstName; 
    private $lastName;

    public  function  construct($anId,  Amount  $amount,  $aFirstName,  $aLastName)
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



