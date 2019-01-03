Doctrine has support for the Serialize LOB pattern. There are plenty of predefined mapping types you can use in order to match Entity attributes with database columns or even tables. One of those mappings is the object type. It maps a SQL CLOB to a PHP object using serialize\(\) and unserialize\(\).

As the Documentation says: “Object Type maps and converts object data based on PHP serialization. If you need to store an exact representation of your object data, you should consider using this type as it uses serialization to represent an exact copy of your object as string in the database. Values retrieved from the database are always converted to PHP’s object type using unserialization or null if no data is present.

This type will always be mapped to the database vendor’s text type internally as there is no way of storing a PHP object representation natively in the database. Furthermore this type requires a SQL column comment hint so that it can be reverse engineered from the database. Doctrine cannot correctly map back this type correctly using vendors that do not support column comments, and will instead fall back to the text type instead. Because the built-in text type of PostgreSQL does not support NULL bytes, the object type will result in unserialization errors. A workaround to this problem is to serialize\(\)/unserialize\(\) and base64\_encode\(\)/base64\_decode\(\) PHP objects and store them into a text field manually.”

Let’s see a possible XML mapping for the Product Entity using the object type.

```php
<?xml version="1.0" encoding="utf-8"?>
<doctrine-mapping
xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">
<entity
name="Product" table="products">
<id
name="id" column="id" type="string" length="255">
<generator
strategy="NONE">
</generator>
</id>
<field
name="name" type="string" length="255"
/>
<field
name="price" type="object"
/>
</entity>
</doctrine-mapping>
```



The key addition is the type="object" that tells Doctrine that we are now going to be using an

object mapping. Let’s now see how we could create and persist an Product entity using Doctrine.

```
// ...
$em->persist($product);
$em->flush($product);
```

Let’s check that if we now fetch our Product Entity from the database it is returned in an expected state.

```php
// ...
$repository  =  $em->getRepository('Ddd\\Domain\\Model\\Product');
$item = $repository->find(1); var_dump($item);

/*
class Ddd\Domain\Model\Product#177 (3) { protected $productId =>
int(1)
protected $name =>
string(41) "Domain-Driven Design in PHP" protected $money =>
class Ddd\Domain\Model\Money#174 (2) { private $amount =>
string(3) "100" private $currency =>
class Ddd\Domain\Model\Currency#175 (1) { private $isoCode =>
string(3) "USD"
}
}
}
*/
```



Last, but not least, as the Doctrine documentation states: “Object types are compared by reference, not by value. Doctrine updates this value if the reference changes and therefore behaves as if these objects are immutable value objects.” Check the Doctrine Basic Mapping Types reference¹⁷ for more information.

This approach suffers from the same refactoring issues as did the ad-hoc ORM. The object mapping type is internally using serialize/unserialize. What about instead using our own serialization?



