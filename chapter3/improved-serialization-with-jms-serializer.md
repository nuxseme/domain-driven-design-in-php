serialize/unserialize native PHP strategies have a problem when dealing with class and namespace refactoring. One alternative is use your own serialization mechanism, for example, concatenating the amount, a one character separator such as “\|” and the currency ISO code. However, there is another better favored approach, using an open-source serializer library such as JMS Serializer¹⁴. Let’s see an example of applying it for serializing a Money object.

序列化/反序列化原生PHP策略在处理类和名称空间重构时存在问题。一种替代方法是使用您自己的序列化机制，例如，连接数量、一个字符分隔符\(如“\|”\)和货币ISO代码。然而,有另一个更好的支持方法,使用一个开源的序列化器库,如JMS序列化器¹⁴。让我们看一个将其用于序列化Money对象的示例。

```php
$myMoney = new Money( 999,
    new Currency('USD')
);

$serializer = JMS\Serializer\SerializerBuilder::create()->build();
$jsonData  =  $serializer->serialize($myMoney,  'json');

In order to unserialize the object, the process is straight forward.

$serializer = JMS\Serializer\SerializerBuilder::create()->build();
// ...
$myMoney = $serializer->deserialize($jsonData, 'Ddd\Domain\Model\Money', 'json');
```

With this example, you can refactor your Money class without having to update your database. JMS Serializer can be used in many different scenarios, for example, when working with REST APIs. An important feature is the ability to specify what attributes of an object should be omitted in the serialization process, a password, for example.

Check the Mapping Reference¹⁵ and the Cookbook¹⁶ for more information. JMS Serializer is a must in any DDD project.

通过这个例子，您可以重构Money类，而不必更新数据库。JMS序列化器可以在许多不同的场景中使用，例如，在使用REST api时。一个重要的特性是能够指定在序列化过程中应该忽略对象的哪些属性，例如密码。

检查映射参考¹⁵和食谱¹⁶获得更多信息。JMS序列化器在任何DDD项目中都是必须的。

