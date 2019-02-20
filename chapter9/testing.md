You’ll see a common pattern while writing your tests. Building entities and complex aggregates could be a very tedious and repetitive process. Complexity and duplication will start creeping in your test suite.

在编写测试时，您将看到一个常见的模式。构建实体和复杂的聚合可能是一个非常繁琐和重复的过程。复杂性和重复将开始在您的测试套件中蔓延。

考虑以下实体

Consider the following entity

```php
class Author
{
    private  $username; private $email; private $fullName;

    public function construct( Username  $aUsername, FullName $aFullName, Email $anEmail
    ) {
        $this->username  =  $aUsername;
        $this->email = $anEmail;
        $this->fullName = $aFullName;
    }

//...
}
```

Included in some test, somewhere in the system

包含在一些测试中，在系统的某个地方

```php
class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $author = new Author(
            new Username('johndoe'),
            new FullName('John', 'Doe'),
            new Email('john@doe.com')
        );

//do something with author
    }
}
```

Services inside boundaries share concepts like entities, aggregates and value objects imagine the clutter of repeating the same building logic over and over across your tests. As we will see, extracting the building logic out of our tests comes very handy and prevents duplication.

边界内的服务共享实体、聚合和值对象等概念，想象一下在测试中重复相同构建逻辑的混乱情况。正如我们将看到的，从测试中提取构建逻辑非常方便，可以防止重复。

