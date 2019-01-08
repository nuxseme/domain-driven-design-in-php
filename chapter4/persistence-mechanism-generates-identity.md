Usually, the simplest way is to let the persistence mechanism to generate the identity because the vast major of persistence mechanisms supports some kind of identity generation, like MySQL’s AUTO\_INCREMENT attribute or Oracle’s/Postgres sequences. This, although simple, have a major drawback: We  won’t have the identity of the entity until we persist it. So to some degree, if  we are going with persistence mechanism generated identities we will couple the identity operation with the underlying persistence store.



```php
CREATE TABLE `orders` (
`id` int(11) NOT NULL auto_increment,
`amount`  decimal(10,  5)  NOT  NULL,
`first_name`  varchar(100)  NOT  NULL,
`last_name`  varchar(100)  NOT  NULL, PRIMARY KEY (`id`)
) ENGINE=InnoDB;

```

And then we might consider this code

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



