Redis⁷ is an in-memory blazing-fast key-value that can be used as a cache or store. Depending on the circumstances we could consider using Redis as a store for our Aggregates. To get started, make sure you have a PHP client to connect to Redis. A good one is Predis⁸.

Redis是一种在内存中快速启动的键值，可以用作缓存或存储。根据情况，我们可以考虑使用Redis作为聚合的存储。首先，确保您有一个PHP客户机要连接到Redis。Predis是一个好例子。

composer require predis/predis:~1.0

```php
namespace Infrastructure\Persistence\Redis;
use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;
use Predis\Client;
class RedisPostRepository implements PostRepository
{
    private $client;
    public function __construct(Client $client)
    {
        $this->client = $client;
    }
    public function save(Post $aPost)
    {
        $this->client->hset('posts', (string) $aPost->id(), serialize($aPost));
    }
    public function remove(Post $aPost)
    {
        $this->client->hdel('posts', (string) $aPost->id());
    }
    public function postOfId(PostId $anId)
    {
        if ($data = $this->client->hget('posts', (string) $anId)) {
            return unserialize($data);
        }
        return null;
    }
    public function latestPosts(\DateTime $sinceADate)
    {
        $latest = $this->filterPosts(
            function(Post $post) use ($sinceADate) {
                return $post->createdAt() > $sinceADate;
            }
        );
        $this->sortByCreatedAt($latest);
        return array_values($latest);
    }
    private function filterPosts(callable $fn)
    {
        return array_filter(array_map(function($data) {
            return unserialize($data);
        }, $this->client->hgetall('posts')), $fn);
    }
    private function sortByCreatedAt(&$posts)
    {
        usort($posts, function(Post $a, Post $b) {
            if ($a->createdAt() == $b->createdAt()) {
                return 0;
            }
            return ($a->createdAt() < $b->createdAt()) ? -1 : 1;
        });
    }
    public function nextIdentity()
    {
        return new PostId();
    }
}
```



