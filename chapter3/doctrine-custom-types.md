Another option is to handle the Value Object persistence using a Doctrine Custom Type. A Custom Type adds to Doctrine a new mapping type that describes a custom transformation between an Entity field and the database representation to persist it.

As the Doctrine documentation explains “just redefining how database types are mapped to all the existing Doctrine types is not at all that useful. You can define your own Doctrine Mapping Types by extending Doctrine\DBAL\Types\Type. You are required to implement 4 different methods to get this working.”

With the object type, the serialization step includes information, such as the class, which makes it quite difficult to safely refactor our code. Let’s try to improve on this solution. Think about a custom serialization process that could solve the problem. One such way could be to persist the Money Value Object as a string in the database encoded in “amount\|isoCode” format?

另一个选项是使用Doctrine自定义类型处理值对象持久性。自定义类型向Doctrine添加了一个新的映射类型，该映射类型描述实体字段与数据库表示之间的自定义转换，以持久化实体字段。

正如Doctrine文档所解释的那样，“仅仅重新定义如何将数据库类型映射到所有现有的Doctrine类型一点用都没有。”您可以通过扩展Doctrine\DBAL\Types\Type来定义自己的Doctrine映射类型。你需要实现4种不同的方法来实现它。

对于对象类型，序列化步骤包括信息，比如类，这使得安全地重构代码非常困难。让我们试着改进这个解决方案。考虑一个可以解决这个问题的定制序列化过程。其中一种方法是将Money Value对象作为字符串保存在以“amount\|isoCode”格式编码的数据库中?

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

使用Doctrine，您需要注册所有自定义类型。通常使用EntityMan- agerFactory来集中创建EntityManager。您也可以在引导应用程序时执行此步骤。

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

现在，我们需要在映射中指定要使用自定义类型。

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
>
> 由于XML映射文件头部中的XSD模式验证，大多数IDE为映射定义中出现的所有元素和属性提供了自动完成功能。

Let’s check the database how the price was persisted using this approach.

让我们检查数据库如何使用这种方法来持久化价格。

```php
mysql> select * from products \G
***************************  1.  row  ***************************  id: 1
name: Domain-Driven Design in PHP price: 999|USD
1 row in set (0.00 sec)
```

This approach is an improvement on the one before in terms of future refactoring, however, searching capabilities remain limited due to the format of the column. With the Doctrine Custom types you can improve the situation a little, but still not the best option for building your DQL queries. Check the Doctrine Custom Mapping Types reference¹⁸ for more information.

在未来的重构方面，这种方法比以前的方法有所改进，但是，由于列的格式，搜索功能仍然有限。使用Doctrine自定义类型，您可以稍微改善这种情况，但仍然不是构建DQL查询的最佳选择。检查原则定义映射类型参考¹⁸获得更多信息。

> Time to discuss
>
> Think and discuss with a peer how would you create a Doctrine Custom Type using JMS to serialize and unserialize a Value Object.
>
> 与同行一起思考和讨论如何使用JMS创建一个Doctrine自定义类型来序列化和反序列化一个值对象



