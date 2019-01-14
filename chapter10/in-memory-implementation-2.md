As an example, if we wanted to replicate the latestPosts query method in our PostRepository by

using a Specification for an in-memory implementation



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





```
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



```php
$latestPosts = $postRepository->query(
    new InMemoryLatestPostSpecification(new \DateTime('-24'))
);
```



