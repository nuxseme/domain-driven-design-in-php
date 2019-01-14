Setting up a fully persistent Repository implementation can be too complex, and result in slow execution. You should care about keeping your tests fast. Going through the whole database setup and querying will slow you down enormously.

Having an in-memory implementation could help delaying persistence decisions until the end.

We could test it in the same manner we did before but this time with a full-featured fast and simple in-memory implementation.

```php
class MyServiceTest extends \PHPUnit_Framework_TestCase
{
    private $service;
    public function setUp()
    {
        $this->service = new MyServiceTest(new InMemoryPostRepository());
    }
}
```



