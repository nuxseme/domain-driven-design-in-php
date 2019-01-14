As Uncle Bob wrote in Screaming Architecture²



A good software architecture allows decisions about frameworks, databases, web- servers, and other environmental issues and tools, to be deferred and delayed. A good architecture makes it unnecessary to decide on Rails, or Spring, or Hibernate, or Tomcat or MySql, until much later in the project. A good architecture makes it easy to change your mind about those decisions too. A good architecture emphasizes the use-cases and decouples them from peripheral concerns.



At the early stages of your application, a fast in-memory implementation could come in handy. It is something you could use to mature other parts of your system allowing you to delay database decisions to the right moment. An in-memory repository is simple, fast and easy to implement.

For our Post repository an in-memory hash-map is enough to provide all the functionality we need.



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



