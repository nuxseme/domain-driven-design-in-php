As Uncle Bob wrote in Screaming Architecture²

A good software architecture allows decisions about frameworks, databases, web- servers, and other environmental issues and tools, to be deferred and delayed. A good architecture makes it unnecessary to decide on Rails, or Spring, or Hibernate, or Tomcat or MySql, until much later in the project. A good architecture makes it easy to change your mind about those decisions too. A good architecture emphasizes the use-cases and decouples them from peripheral concerns.

At the early stages of your application, a fast in-memory implementation could come in handy. It is something you could use to mature other parts of your system allowing you to delay database decisions to the right moment. An in-memory repository is simple, fast and easy to implement.

一个好的软件架构允许对框架、数据库、web服务器以及其他环境问题和工具的决策被推迟或延迟。一个好的体系结构使得在项目后期之前，不必再决定Rails、Spring、Hibernate、Tomcat或MySql。好的架构也可以很容易地改变您对这些决策的想法。好的架构强调用例，并将它们与外围关注点分离。

在应用程序的早期阶段，快速内存实现可能会派上用场。您可以使用它来成熟系统的其他部分，从而将数据库决策延迟到正确的时刻。内存存储库简单、快速且易于实现。

For our Post repository an in-memory hash-map is enough to provide all the functionality we need.

对于我们的Post存储库，内存中的hashmap足以提供我们需要的所有功能。

```php
namespace Infrastructure\Persistence\InMemory;

use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;

class InMemoryPostRepository implements PostRepository
{
    private $posts = [];

    public function add(Post $aPost)
    {
        $this->posts[$aPost->id()->id()] = $aPost;
    }

    public function remove(Post $aPost)
    {
        unset($this->posts[$aPost->id()->id()]);
    }

    public function postOfId(PostId $anId)
    {
        if (isset($this->posts[$anId->id()])) {
            return $this->posts[$anId->id()];
        }

        return null;
    }

    public function latestPosts(\DateTime $sinceADate)
    {
        return $this->filterPosts(
            function(Post $post) use ($sinceADate) {
                return $post->createdAt() > $sinceADate;
            }
        );
    }

    private function filterPosts(callable $fn)
    {
        return array_values(array_filter($this->posts, $fn));
    }
    public function nextIdentity()

    {
        return new PostId();
    }
}
```



