An Object Mother³ is a catchy name for a factory that creates fixed fixtures for your tests.

Following the previous example, we could extract the duplicated logic to an Object Mother so it could be reused across tests.

对象母亲³的工厂是一个朗朗上口的名字为测试创建固定装置。

按照前面的示例，我们可以将重复的逻辑提取到对象母亲，这样就可以跨测试重用它。

```php
class AuthorObjectMother
{
    public static function createOne()
    {
        return new Author(
            new Username('johndoe'),
            new FullName('John', 'Doe'),
            new Email('john@doe.com')
        );
    }
}


class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $author = AuthorObjectMother::createOne();

//do something with author
    }
}
```

You’ll notice that the more tests and situations you have, the more methods the factory will have.

As Object Mothers are not very flexible, they tend to grow in complexity quickly. There is a more flexible alternative for your tests.

您将注意到，您拥有的测试和情况越多，工厂拥有的方法就越多。

由于对象母亲不是很灵活，它们的复杂性往往会迅速增加。对于您的测试，有一个更灵活的替代方案

