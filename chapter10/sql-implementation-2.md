A standard Specification works well for in-memory implementations. However, as we do not

pre-load all the domain objects in-memory for a SQL implementation, we need a more specific

specification for these cases.

Astandard规范可以很好地用于内存实现。然而，我们没有为SQL实现预加载内存中的所有域对象，我们需要一个更具体的这些情况的规范。

```php
namespace Infrastructure\Persistence\Sql;
interface SqlPostSpecification
{
    /**
     * @return string
     */
    public function toSqlClauses();
}
```

The SQL implementation for this specification could look like

该规范的SQL实现如下所示

```php
namespace Infrastructure\Persistence\Sql;
class SqlLatestPostSpecification implements SqlPostSpecification
{
    private $since;
    public function __construct(\DateTime $since)
    {
        $this->since = $since;
    }
    public function toSqlClauses()
    {
        return "created_at > '" . $this->since->format('Y-m-d H:i:s') . "'";
    }
}
```

And how to query an SQL Post repository implementation

以及如何查询SQL Post存储库实现

```php
class SqlPostRepository implements PostRepository
{
    /**
     * @param SqlPostSpecification $specification
     *
     * @return Post[]
     */
    public function query($specification)
    {
        return $this->retrieveAll(
            'SELECT * FROM posts WHERE ' . $specification->toSqlClauses()
        );
    }
    private function retrieveAll($sql, array $parameters = [])
    {
        $st = $this->pdo->prepare($sql);
        $st->execute($parameters);
        return array_map(function ($row) {
            return $this->buildPost($row);
        }, $st->fetchAll(\PDO::FETCH_ASSOC));
    }
}
```



