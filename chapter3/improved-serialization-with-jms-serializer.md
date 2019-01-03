serialize/unserialize native PHP strategies have a problem when dealing with class and namespace refactoring. One alternative is use your own serialization mechanism, for example, concatenating the amount, a one character separator such as “\|” and the currency ISO code. However, there is another better favored approach, using an open-source serializer library such as JMS Serializer¹⁴. Let’s see an example of applying it for serializing a Money object.

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



