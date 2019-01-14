An Object Mother³ is a catchy name for a factory that creates fixed fixtures for your tests.

Following the previous example, we could extract the duplicated logic to an Object Mother so it could be reused across tests.





```
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



