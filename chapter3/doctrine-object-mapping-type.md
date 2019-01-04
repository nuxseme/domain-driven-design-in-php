Doctrine has support for the Serialize LOB pattern. There are plenty of predefined mapping types you can use in order to match Entity attributes with database columns or even tables. One of those mappings is the object type. It maps a SQL CLOB to a PHP object using serialize\(\) and unserialize\(\).

As the Documentation says: “Object Type maps and converts object data based on PHP serialization. If you need to store an exact representation of your object data, you should consider using this type as it uses serialization to represent an exact copy of your object as string in the database. Values retrieved from the database are always converted to PHP’s object type using unserialization or null if no data is present.

This type will always be mapped to the database vendor’s text type internally as there is no way of storing a PHP object representation natively in the database. Furthermore this type requires a SQL column comment hint so that it can be reverse engineered from the database. Doctrine cannot correctly map back this type correctly using vendors that do not support column comments, and will instead fall back to the text type instead. Because the built-in text type of PostgreSQL does not support NULL bytes, the object type will result in unserialization errors. A workaround to this problem is to serialize\(\)/unserialize\(\) and base64\_encode\(\)/base64\_decode\(\) PHP objects and store them into a text field manually.”

Let’s see a possible XML mapping for the Product Entity using the object type.

Doctrine支持序列化LOB模式。为了将实体属性与数据库列甚至表匹配，您可以使用许多预定义的映射类型。其中一个映射是对象类型。它使用serialize\(\)和unserialize\(\)将SQL CLOB映射到PHP对象。

正如文档中所说:“对象类型映射和转换基于PHP序列化的对象数据。如果需要存储对象数据的精确表示，应该考虑使用这种类型，因为它使用序列化将对象的精确副本表示为数据库中的字符串。从数据库检索的值总是使用反序列化转换为PHP的对象类型，如果没有数据，则转换为null。

这种类型总是在内部映射到数据库供应商的文本类型，因为无法在数据库中本地存储PHP对象表示形式。此外，这种类型需要一个SQL列注释提示，以便从数据库对其进行反向工程。Doctrine无法使用不支持列注释的供应商正确地映射回此类型，而是将返回到文本类型。因为PostgreSQL的内置文本类型不支持空字节，所以对象类型将导致反序列化错误。解决这个问题的一个方法是序列化\(\)/unserialize\(\)和base64\_encode\(\)/base64\_decode\(\) PHP对象，并手动将它们存储到一个文本字段中。



让我们看看使用对象类型的产品实体可能的XML映射。

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

关键的添加是type="object"，它告诉Doctrine我们现在将使用an

对象映射。现在让我们看看如何使用Doctrine创建和持久化产品实体。

```
// ...
$em->persist($product);
$em->flush($product);
```

Let’s check that if we now fetch our Product Entity from the database it is returned in an expected state.

让我们检查一下，如果现在从数据库中获取产品实体，它是否以期望的状态返回。

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

最后，但并非最不重要的是，正如Doctrine文档所述:“通过引用而不是通过值来比较对象类型。如果引用发生更改，Doctrine将更新此值，并因此将这些对象视为不可变值对象。“检查原则基本映射类型参考¹⁷获得更多信息。

这种方法遇到的重构问题与特别ORM相同。对象映射类型在内部使用serialize/unserialize。那么使用我们自己的序列化呢?

