Setting up a fully persistent Repository implementation can be too complex, and result in slow execution. You should care about keeping your tests fast. Going through the whole database setup and querying will slow you down enormously.

Having an in-memory implementation could help delaying persistence decisions until the end.

We could test it in the same manner we did before but this time with a full-featured fast and simple in-memory implementation.

设置完整的持久性存储库实现可能过于复杂，导致执行缓慢。你应该注意保持你的测试速度。完成整个数据库设置和查询将大大降低您的速度。

在内存中实现有助于将持久性决策延迟到最后。

我们可以用与以前相同的方式测试它，但是这次使用了功能齐全的快速和简单的内存实现。

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



