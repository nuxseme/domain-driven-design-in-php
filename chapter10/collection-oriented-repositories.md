Repositories mimic a collection by implementing their common interface characteristics. As a collection it should not leak any intentions of persistence behaviour, such as the notion of saving to a store.

The underlying persistence mechanism has to support for this need. You should not be required to handle changes to the objects over its lifetime. The collection references the most recent changes to the object, meaning that upon each access you get the latest object state.

Repositories implement a concrete collection type, the Set. A Set is a data-structure with the invariant that does not contain duplicate entries. If you try to add an element to a Set that is already present, it will not be added. This is useful in our use-case as each Aggregate has a unique identity that is associated with the Root Entity.

存储库通过实现它们的公共接口特征来模拟集合。作为一个集合，它不应该泄漏任何持久性行为的意图，例如保存到存储的概念。

底层持久性机制必须支持这种需求。不应该要求您在对象的生命周期中处理对象的更改。集合引用对象的最新更改，这意味着每次访问时您都将获得最新的对象状态。

存储库实现一个具体的集合类型，集合。集合是一个数据结构，不包含重复的条目。如果您试图向已经存在的集合中添加元素，则不会添加该元素。这在我们的用例中非常有用，因为每个聚合都有一个与根实体相关联的惟一标识。

If for example we have the following Domain Model

例如，如果我们有以下领域模型

```php
namespace Domain\Model;

class Post
{
    const  EXPIRE_EDIT_TIME  =  120;  //  seconds

    private $id; private $body; private $createdAt;

    public function construct( PostId $anId,
        Body  $aBody,
        \DateTime $createdAt = null
    ) {
        $this->id = $anId;
        $this->body  =  $aBody;
        $this->createdAt  =  $createdAt  ?:  new  \DateTime();
    }

    public function editBody(Body $aNewBody)
    {
        if ($this->editExpired()) {
            throw new \RuntimeException('Edit time expired');
        }

        $this->body  =  $aNewBody;
    }

    private function editExpired()
    {
        $expiringTime  =  $this->createdAt->getTimestamp()  +  self::EXPIRE_EDIT_TIME;

return $expiringTime < time();
}

    public function id()
    {
        return $this->id;
    }

    public function body()
    {
        return $this->body;
    }

    public function createdAt()
    {
        return $this->createdAt;
    }
}
class Body
{
    const  MIN_LENGTH  =  3;
    const  MAX_LENGTH  =  250;

    private $content;

    public function construct($content)
    {
        $this->setContent(trim($content));
    }

    private function setContent($content)
    {
        $this->assertNotEmpty($content);
        $this->assertFitsLength($content);

        $this->content = $content;
    }

    private function assertNotEmpty($content)
    {
        if (empty($content)) {
            throw new \DomainException('Empty body');
        }
    }

    private function assertFitsLength($content)
    {
        if  (strlen($content)  <  self::MIN_LENGTH)  {
            throw new \DomainException('Body is too sort');
        }

        if  (strlen($content)  >  self::MAX_LENGTH)  {
            throw new \DomainException('Body is too long');
        }
    }

    public function content()
    {
        return $this->content;
    }
}

class PostId
{
    private $id;

    public function construct($id = null)
    {
        $this->id  =  $id  ?:  uniqid();
    }

    public function id()
    {
        return $this->id;
    }

    public function equals(PostId $anId)
    {
        return $this->id === $anId->id();
    }
}
```

If we wished to persist this Post entity, a simple in-memory Post Repository could be created like the following

如果我们希望持久化这个Post实体，可以像下面这样创建一个简单的内存Post存储库

```php
class SimplePostRepository
{
    private $post = [];

    public function add(Post $aPost)
    {
        $this->posts[(string)$aPost->id()] = $aPost;
    }

    public function postOfId(PostId $anId)
    {
        if (isset($this->posts[(string) $anId])) {
            return $this->posts[(string) $anId];
        }

        return null;
    }
}
```

And, as you would expect it is handled as a collection

并且，正如您所期望的，它作为一个集合来处理

```php
$id = new PostId();
$repository = new SimplePostRepository();
$repository->add(new Post($id, 'Random content'));

// later ...
$post    =  $repository->postOfId($id);
$post->editBody('Updated content');

// even later ...
$post = $repository->postOfId($id); assert('Updated content' === $post->body());
```

As you can see, from the collections point of view there is no need for a save method in the repository. Changes affecting the object are correctly handled by the underlying persistence layer.

The first step is to define a collection-like interface for your repository. The interface needs to define the usual collection methods, as following.

如您所见，从集合的角度来看，存储库中不需要保存方法。影响对象的更改由底层持久性层正确处理。

第一步是为存储库定义类似集合的接口。接口需要定义通常的集合方法，如下所示。

```php
interface PostRepository
{
    public function add(Post $aPost);
    public function addAll(array $posts);
    public function remove(Post $aPost);
    public function removeAll(array $posts);
// ...
}
```

The interface definition should be placed in the same module that the Aggregate uses to store.

Sometimes remove does not provide true Aggregate removal. There are times where you need to keep the information for legal purposes or business intelligence. In those cases, you can instead mark the Aggregate as disabled or logically removed. The interface could be updated accordingly, removing the removal methods or providing disable behaviour in the repository.

Another important part of repositories are the finder methods such as.

接口定义应该放在聚合用于存储的模块中。

有时移除并不提供真正的骨料移除。有时您需要将这些信息用于法律目的或商业智能。在这些情况下，您可以将聚合标记为禁用或逻辑删除。可以相应地更新接口，删除删除方法或在存储库中提供禁用行为。

存储库的另一个重要部分是查找器方法，例如。

```php
interface PostRepository
{

    /**
    @return Post
     */
    public function postOfId(PostId $anId);

    /**
    @return Post[]
     */
    public function latestPosts(\DateTime $sinceADate);
}
```

And, to retrieve the globally unique id for a Post, a logical place to include it is

并且，要检索Post的全局惟一id，包含它的逻辑位置是

```php
interface PostRepository
{

    /**
    @return PostId
     */
    public function nextIdentity();
}
```

The code responsible for building up each Post instance calls nextIdentity to get the unique identifier PostId.

负责构建每个Post实例的代码调用nextIdentity以获得唯一标识符PostId。

```php
$post = new Post($postRepository->nextIdentity(), $body);
```

Some developers favour placing the implementation close to the interface definition, as a sub- package of the module. However, because we want a clear separation of concerns, we recommend to place it inside the infrastructure layer instead.

一些开发人员倾向于将实现放在接口定义附近，作为模块的子包。但是，因为我们想要清楚地分离关注点，所以我们建议将其放在基础结构层中。

