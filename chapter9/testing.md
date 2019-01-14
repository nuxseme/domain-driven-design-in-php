You’ll see a common pattern while writing your tests. Building entities and complex aggregates could be a very tedious and repetitive process. Complexity and duplication will start creeping in your test suite.

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



