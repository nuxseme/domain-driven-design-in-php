Using Specifications in our services might be the best example to illustrate how to use factories within our services.

Consider the following service example. Given a request from the outside world, we want to build a feed based on the latest Posts added to the system.



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



```php
namespace Domain\Model;

interface PostSpecificationFactory
{
    public function createLatestPosts(\DateTime $since);
}
```



Then we need to create factories for each PostRepository implementations. As an example, a Factory for the in-memory PostRepository implementation could be like

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



