Following the previous example, we mentioned that different concerns should be split up. In order

to do so, all layers should be identified in our original tangled code. Through this process we need

to pay special attention to the code conforming to the Model layer, which will be the beating heart

of the application.

> 不同的概念需要分离。为了如此，所有的层需要被标识在复杂的代码中。在这个过程中，我们可以给予模型层更多的关注，以驱动应用

```php
class Post
{
    private $title;
    private $content;
    public static function writeNewFrom($title, $content)
    {
        return new static($title, $content);
    }
    private function __construct($title, $content)
    {
        $this->setTitle($title);
        $this->setContent($content);
    }
    private function setTitle($title)
    {
        $this->assertNotEmpty($title);
        $this->title = $title;
    }
    private function setContent($content)
    {
        $this->assertNotEmpty($title);
        $this->content = $content;
    }
}
```

```php
class PostRepository
{
    private $db;
    public function __construct()
    {
        $this->db = new PDO(
            'mysql:host=localhost;dbname=my_database',
            'a_username',
            '4_p4ssw0rd',
            [
                PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8',
            ]
        );
    }
    public static function add(Post $post)
    {
        $this->db->beginTransaction();
        try {
            $stm = $db->prepare(
                'INSERT INTO posts (title, content) VALUES (?, ?)'
            );
            $stm->exec([
                $post->getTitle(),
                $post->getContent(),
            ]);
            $db->commit();
        } catch (Exception $e) {
            $db->rollback();
            throw new UnableToCreatePostException($e);
        }
    }
}
```

The Model layer is now defined by a Post class and a PostRepository class. The Post class represents

a blog post and the PostRepository class represents the whole collection of blog posts available.

Additionally, another layer inside the Model is needed, a layer that coordinates and orchestrates the

domain model behaviour: the Application Layer.

> 现在模型层通过Post和 PostRepository类来定义。Post代表一个博客文章模型，PostRepository代码整个可用的文章模型。另外，需要一个控制层来协调和编排模型层的行为

```php
class PostService
{
    public function createPost($title, $content)
    {
        $post = Post::writeNewFrom($title, $content);
        (new PostRepository())->add($post);
        return $post;
    }
}
```

The PostService is what is known as an Application Service and its purpose is to orchestrate and

organize the domain behaviour. In other words, the Application services are the ones that make

things happen and they are the direct clients of a Domain Model. No other type of object should

be able to directly talk to the internal layers of the Model layer.

> PostService 作为应用服务层，是为了组织和管理领域的行为。换句话说，应用层服务直接调用领域模型。其他类型的对象不可以直接调用内部的模型层

---

* 分层  区域自治
* 责任隔离 
* 关系约束



