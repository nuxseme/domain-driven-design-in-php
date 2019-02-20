As an example, if we wanted to replicate the latestPosts query method in our PostRepository by

using a Specification for an in-memory implementation

例如，如果我们想复制PostRepository中的latestPosts查询方法

为内存中的实现使用规范

```php
namespace Infrastructure\Persistence\InMemory;
use Domain\Model\Post;
interface InMemoryPostSpecification
{
    /**
     * @return boolean
     */
    public function specifies(Post $aPost);
}
```

The in-memory implementation for the latestPosts behaviour could be as follows

latestPosts行为的内存实现可以如下所示

```php
namespace Infrastructure\Persistence\InMemory;
use Domain\Model\Post;
class InMemoryLatestPostSpecification implements InMemoryPostSpecification
{
    private $since;
    public function __construct(\DateTime $since)
    {
        $this->since = $since;
    }
    public function specifies(Post $aPost)
    {
        return $aPost->createdAt() > $this->since;
    }
}
```

The query method for our repository implementation could look as follows

存储库实现的查询方法如下所示

```php
class InMemoryPostRepository implements PostRepository
{
    /**
     * @param InMemoryPostSpecification $specification
     *
     * @return Post[]
     */
    public function query($specification)
    {
        return $this->filterPosts(
            function(Post $post) use ($specification) {
                return $specification->specifies($post);
            }
        );
    }
}
```

Retrieving all the latest posts from the repository is as simple as creating a tailored instance of the  above implementation

从存储库中检索所有最新帖子就像创建上述实现的定制实例一样简单

```php
$latestPosts = $postRepository->query(
    new InMemoryLatestPostSpecification(new \DateTime('-24'))
);
```



