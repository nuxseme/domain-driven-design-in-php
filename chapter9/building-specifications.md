Using Specifications in our services might be the best example to illustrate how to use factories within our services.

Consider the following service example. Given a request from the outside world, we want to build a feed based on the latest Posts added to the system.

在服务中使用规范可能是说明如何在服务中使用工厂的最佳示例。

考虑下面的服务示例。给定外部世界的请求，我们希望基于添加到系统中的最新帖子构建提要。

```php
namespace Application\Service;

use Domain\Model\Post;
use Domain\Model\PostRepository;

class  LatestPostsFeedService
{
    private $postRepository;

    public function construct(PostRepository $postRepository)
    {
        $this->postRepository = $postRepository;
    }

    /**
     * @param LatestPostsFeedRequest $request
     */
    public function execute($request)
    {
        $posts = $this->postRepository->latestPosts($request->since);

        return array_map(function(Post $post) {
            return [
                'id' => $post->id()->id(),
                'content' => $post->body()->content(), 'created_at' => $post->createdAt()
            ];
        }, $posts);
    }
}
```

Finder methods in Repositories like latestPosts have some limitations as they keep adding complexity to our repositories indefinitely. As we discussed in Repositories chapter, Specifications are a better approach.

Lucky for us, we have a nice query method in our PostRepository that works with Specifications.

像latestpost这样的存储库中的查找器方法有一些限制，因为它们会无限期地增加存储库的复杂性。正如我们在存储库一章中讨论的，规范是一种更好的方法。

幸运的是，在我们的PostRepository中有一个很好的查询方法，它符合规范。

```php
class LatestPostsFeedService
{
//...

    public function execute($request)
    {
//...

        $posts = $this->postRepository->query($specification);

//...
    }
}
```

Using a concrete implementation for the Specification is a bad idea

为规范使用具体的实现不是一个好主意

```php
class LatestPostsFeedService
{
//...

    public function execute($request)
    {
        $posts = $this->postRepository->query(
            new SqlLatestPostSpecification($request->since)
        );

//...
    }
}
```

Coupling our high-level application service with a low-level Specification implementation mixes layers and breaks separation of concerns. In addition, its a pretty bad way of coupling our service to a concrete infrastructure implementation. There’s no way you could use this service out of the SQL persistence solution. What if we want to test our service with an in-memory implementation?

The solution to this problem is to decouple Specification creation from the service itself by using the Abstract Factory pattern².

Abstract Factory offers the interface for creating a family of related objects, without explicitly specifying their classes.

As we might have multiple Specification implementations we need to create an interface for the factory first.

将我们的高级应用程序服务与低级规范实现耦合起来会混合层并打破关注点的分离。此外，将我们的服务与具体的基础设施实现耦合是一种非常糟糕的方式。您不可能在SQL持久性解决方案之外使用此服务。如果我们想用内存实现来测试服务，该怎么办?

此问题的解决方案是脱钩规范创建服务本身通过使用抽象工厂模式²。

抽象工厂提供了创建一系列相关对象的接口，而不需要显式地指定它们的类。

由于我们可能有多个规范实现，我们需要首先为工厂创建一个接口。

```php
namespace Domain\Model;

interface PostSpecificationFactory
{
    public function createLatestPosts(\DateTime $since);
}
```

Then we need to create factories for each PostRepository implementations. As an example, a Factory for the in-memory PostRepository implementation could be like

然后我们需要为每个PostRepository实现创建工厂。例如，内存中PostRepository实现的工厂可能是这样的

```php
namespace Infrastructure\Persistence\InMemory;

use Domain\Model\PostSpecificationFactory;

class InMemoryPostSpecificationFactory implements PostSpecificationFactory
{
    public function createLatestPosts(\DateTime $since)
    {
        return new InMemoryLatestPostSpecification($since);
    }
}
```

Once we have a centralised place for the creation logic, its easy to decouple it from the service

一旦我们有了创建逻辑的集中地，就很容易将其与服务解耦

```php
class LatestPostsFeedService
{
    private  $postRepository;
    private $postSpecificationFactory;

    public function construct( PostRepository $postRepository,
        PostSpecificationFactory $postSpecificationFactory
    ) {
        $this->postRepository = $postRepository;
        $this->postSpecificationFactory = $postSpecificationFactory;
    }

    public function execute($request)
    {
        $posts = $this->postRepository->query(
            $this->postSpecificationFactory->createLatestPosts($request->since)
        );

// ...
    }
}
```

Now unit-testing our service through an in-memory PostRepository implementation is pretty easy

现在，通过内存中的PostRepository实现对我们的服务进行单元测试非常简单

```php
use Domain\Model\Body;
use Domain\Model\Post;
use Domain\Model\PostId;
use Infrastructure\Persistence\InMemory\InMemoryPostRepository;

class LatestPostsFeedServiceTest extends \PHPUnit_Framework_TestCase
{
    /**
     * @var \Infrastructure\Persistence\InMemory\InMemoryPostRepository
     */
    private $postRepository;

    /**
     * @var LatestPostsFeedService
     */
    private $latestPostsFeedService;

    public function setUp()
    {
        $this->latestPostsFeedService = new LatestPostsFeedService(
            $this->postRepository = new InMemoryPostRepository()
        );
    }

    /**
     * @test
     */
    public function shouldBuildAFeedFromLatestsPosts()
    {
        $this->addPost(1,  'first',  '-2  hours');
        $this->addPost(2,  'second',  '-3  hours');
        $this->addPost(3,  'third',  '-5  hours');

        $feed = $this->latestPostsFeedService->execute(
            new LatestPostsFeedRequest(new \DateTime('-4 hours'))
        );

        $this->assertFeedContains([
            ['id'  =>  1,  'content'  =>  'first'],
            ['id'  =>  2,  'content'  =>  'second']
        ], $feed);
    }

    private function addPost($id, $content, $createdAt)
    {
        $this->postRepository->add(new Post(
            new PostId($id),
            new Body($content),
            new \DateTime($createdAt)
        ));
    }

    private function assertFeedContains($expected, $feed)
    {
        foreach ($expected as $index => $contents) {
            $this->assertArraySubset($contents, $feed[$index]);
            $this->assertNotNull($feed[$index]['created_at']);
        }
    }
}
```



