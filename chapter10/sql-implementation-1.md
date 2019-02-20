In a classic example, we could create a simple PDO⁹ implementation for our PostRepository just by using plain SQL queries.

在一个典型的例子中，我们可以使用普通的SQL查询为PostRepository创建一个简单的PDO实现。

```php
namespace Infrastructure\Persistence\Sql;
use Domain\Model\Body;
use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;
class SqlPostRepository implements PostRepository
{
    const DATE_FORMAT = 'Y-m-d H:i:s';
    private $pdo;
    public function __construct(\PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    public function save(Post $aPost)
    {
        $sql = 'INSERT INTO posts (id, body, created_at) VALUES (:id, :body, :cr\
eated_at)';
        $this->execute($sql, [
            'id' => $aPost->id()->id(),
            'body' => $aPost->body()->content(),
            'created_at' => $aPost->createdAt()->format(self::DATE_FORMAT)
        ]);
    }
    private function execute($sql, array $parameters)
    {
        $st = $this->pdo->prepare($sql);
        $st->execute($parameters);
        return $st;
    }
    public function remove(Post $aPost)
    {
        $this->execute('DELETE FROM posts WHERE id = :id', [
            'id' => $aPost->id()->id()
        ]);
    }
    public function postOfId(PostId $anId)
    {
        $st = $this->execute('SELECT * FROM posts WHERE id = :id', [
            'id' => $anId->id()
        ]);
        if ($row = $st->fetch(\PDO::FETCH_ASSOC)) {
            return $this->buildPost($row);
        }
        return null;
    }
    private function buildPost($row)
    {
        return new Post(
            new PostId($row['id']),
            new Body($row['body']),
            new \DateTime($row['created_at'])
        );
    }
    public function latestPosts(\DateTime $sinceADate)
    {
        return $this->retrieveAll('SELECT * FROM posts WHERE created_at > :since\
_date', [
            'since_date' => $sinceADate->format(self::DATE_FORMAT)
        ]);
    }
    private function retrieveAll($sql, array $parameters = [])
    {
        $st = $this->pdo->prepare($sql);
        $st->execute($parameters);
        return array_map(function ($row) {
            return $this->buildPost($row);
        }, $st->fetchAll(\PDO::FETCH_ASSOC));
    }
    public function nextIdentity()
    {
        return new PostId();
    }
    public function size()
    {
        return $this->pdo->query('SELECT COUNT(*) FROM posts')
            ->fetchColumn();
    }
}
```

As we do not have any mapping configuration, it would be very useful to have an initialisation method for the schema within the same class. Things that change together should remain together.

由于我们没有任何映射配置，所以在同一个类中为模式提供一个初始化方法是非常有用的。一起改变的事物应该保持在一起。

```php
class SqlPostRepository implements PostRepository
{
// ...
    public function initSchema()
    {
        $this->pdo->exec(<<<SQL
DROP TABLE IF EXISTS posts;
CREATE TABLE posts (
id CHAR(36) PRIMARY KEY,
body VARCHAR(250) NOT NULL,
created_at DATETIME NOT NULL
)
SQL
        );
    }
}
```



