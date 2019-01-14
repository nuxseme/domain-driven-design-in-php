One of the features used to present Doctrine when it was released was that mapping information can be specified using annotated code

> What’s an annotation?
>
> An annotation is a special form of metadata. In PHP is put under source code comments. For example, PHPDocumentor makes use of annotations to build API information or PHPUnit uses some annotations to specify dataProviders or to provide expectations about exceptions thrown by a piece of code

```php
class SumTest extends PHPUnit_Framework_TestCase {
    /**
     * @dataProvider aMethodName
     */
    public function testAddition() {
// ...
    }
}
```

So in order to map the Order entity to the persistence store first of all the source code for the Order should be modified to add the Doctrine annotations

```php

use Doctrine\ORM\Mapping\Entity;
use Doctrine\ORM\Mapping\Id;
use Doctrine\ORM\Mapping\GeneratedValue;
use Doctrine\ORM\Mapping\Column;

/** @Entity */
class Order
{
    /** @Id @GeneratedValue(strategy="AUTO") */
    private $id;

    /** @Column(type="decimal", precision="10", scale="5") */
    private  $amount;

    /** @Column(type="string") */
    private $firstName;

    /** @Column(type="string") */
    private $lastName;

    public  function construct(Amount  $anAmount,  $aFirstName,  $aLastName)
    {
        $this->amount  =  $anAmount;
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



And then to persist the entity to the persistent store it’s just as easy as

```php
$order = new Order(
    new  Amount(15,  Currency::EUR()), 'AFirstName',
    'ALastName'
);

$entityManager->persist($order);
$entityManager->flush();

```

At a first glance, this code can look simple and this can be an easy way to specify mapping information. But this way comes at a cost. What’s odd about the final code?

First of all, domain concerns are mixed with infrastructure concerns. Order is a domain concept whereas Table, Column and so on are infrastructure concerns.



And so it is, that this entity is tightly coupled to the mapping information specified by the annotations in the source code. If the entity were required to be persisted using another entity manager and with a different mapping metadata, it would not be possible.

Annotations tend to lead to side-effects and tight coupling. So it would be better to not use them.

So what’s the best way to specify mapping information? The best way is the one that allows to separate the mapping information from the entity itself. And this can be achieved by using XML mapping, YAML mapping or PHP mapping. In this book we are going to cover XML mapping.



