Another option is to handle the Value Object persistence using a Doctrine Custom Type. A Custom Type adds to Doctrine a new mapping type that describes a custom transformation between an Entity field and the database representation to persist it.

As the Doctrine documentation explains “just redefining how database types are mapped to all the existing Doctrine types is not at all that useful. You can define your own Doctrine Mapping Types by extending Doctrine\DBAL\Types\Type. You are required to implement 4 different methods to get this working.”

With the object type, the serialization step includes information, such as the class, which makes it quite difficult to safely refactor our code. Let’s try to improve on this solution. Think about a custom serialization process that could solve the problem. One such way could be to persist the Money Value Object as a string in the database encoded in “amount\|isoCode” format?



```php
use Ddd\Domain\Model\Currency;
use Ddd\Domain\Model\Money;
use Doctrine\DBAL\Types\TextType;
use Doctrine\DBAL\Platforms\AbstractPlatform;

class MoneyType extends TextType
{
    const MONEY = 'money';

    public function convertToPHPValue($value, AbstractPlatform $platform)
    {
        $value = parent::convertToPHPValue($value, $platform);

        $value = explode('|', $value);
        return new Money(
            $value[0],
            new  Currency($value[1])
        );
    }

    public function convertToDatabaseValue($value, AbstractPlatform $platform)
    {
        return implode('|',
            [
                $value->amount(),
                $value->currency()->isoCode()
            ]
        );
    }

    public function getName()
    {
        return self::MONEY;
    }
}
```

Using Doctrine you are required to register all Custom Types. It is common to use an EntityMan- agerFactory that centralizes this EntityManager creation. You could alternatively do this step in bootstrapping your application.

```php
use Doctrine\DBAL\Types\Type;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Tools\Setup;

class EntityManagerFactory
{
    public function build()
    {
        Type::addType('money',
            'Ddd\\Infrastructure\\Persistence\\Doctrine\\Type\\MoneyType'
        );

        return EntityManager::create(
            array(
                'driver'   => 'pdo_mysql',
                'user'     => 'root',
                'password' => '',
                'dbname'   => 'ddd',
            ),
            Setup::createXMLMetadataConfiguration([DIR . '/config'],
                true
            )
        );
    }
}
```

Now, we need to specify in the mapping that we want to use our Custom Type.

```php

<?xml version="1.0" encoding="utf-8"?>
<doctrine-mapping>
<entity
name="Product" table="product">
<!-- ... -->
<field
name="price" type="money"
/>
</entity>
</doctrine-mapping>
```



> Why use XML mapping?
>
> Thanks to the XSD schema validation in the headers of the XML mapping file, most IDE’s provide auto-complete functionality for all the elements and attributes present in the mapping definition.



Let’s check the database how the price was persisted using this approach.



```php
mysql> select * from products \G
***************************  1.  row  ***************************  id: 1
name: Domain-Driven Design in PHP price: 999|USD
1 row in set (0.00 sec)
```

This approach is an improvement on the one before in terms of future refactoring, however, searching capabilities remain limited due to the format of the column. With the Doctrine Custom types you can improve the situation a little, but still not the best option for building your DQL queries. Check the Doctrine Custom Mapping Types reference¹⁸ for more information.



> Time to discuss
>
> Think and discuss with a peer how would you create a Doctrine Custom Type using JMS to serialize and unserialize a Value Object.



