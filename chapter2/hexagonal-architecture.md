Following on with the blog example application, the first concept we need is a Port where the outside

world could talk to the application. For this case, we will use an HTTP Port and its corresponding

Adapter. The outside will use the port to send messages to the application. The example was using

a database to store the whole collection of blog posts so, in order to allow the application retrieve

blog posts from the database, a Port is needed

下面是一个blog的例子，需要一个端口跟外部的应用沟通。在这个案例中，我们使用HTTP Port来适配。外部通过这个端口发送消息给应用。这个例子使用数据库来存储所有的blog集合。为了使应用可以检索所有的blog,需要一个面向数据库的端口

```php
interface PostRepository
{
    public function byId(PostId $id);
    public function add(Post $post);
}
```

This interface states the Port by which the application will retrieve information about blog posts,

and it will be located in the Domain Layer. Now, an Adapter for this Port is needed. The Adapter is

in charge of defining the way in which the blog posts will be retrieved using a specific technology

这个接口说明了检索blog的端口，它将会被置于领域层。现在需要一个适配器来匹配这个端口。这个适配器用来定义blog被检索的特定的方式

```php
class PDOPostRepository implements PostRepository
{
    private $pdo;
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    public function byId(PostId $id)
    {
        $stm = $this->db->prepare(
            'SELECT * FROM posts WHERE id = ?'
        );
        $stm->execute([$id->id()]);
        return recreateFrom($stm->fetch());
    }
    public function add(Post $post)
    {
        $stm = $db->prepare(
            'INSERT INTO posts (title, content) VALUES (?, ?)'
        );
        $stm->exec([
            $post->getTitle(),
            $post->getContent(),
        ]);
    }
}
```

Once we have the Port and its Adapter defined, the last step to do is to refactor the PostService

class so that it uses them. And this can be easily achieved by using Dependency Injection⁸

一旦我们定义了端口和适配器。最后一步就是重构PostService来使用它们。使得依赖注入可以很容易

```php
class PostService
{
    private $postRepository;
    public function __construct(PostRepository $postRepository)
    {
        $this->postRepository = $postRepository;
    }
    public function createPost($title, $content)
    {
        $post = Post::writeNewFrom($title, $content);
        $this->postRepository->add($post);
        return $post;
    }
}
```

This is just a simple example of Hexagonal Architecture. It is a flexible architecture that promotes

separation of concerns like layered architecture and symmetry in that there is an inside application

that communicates using ports with the outside. From now on, this will be the foundational

architecture used to build and explain CQRS and Event Sourcing.

For a deeper insight about this architecture, you can checkout the appendix with a detailed

example, explaining advanced topics like transactionability and other cross cutting concerns.

这是使用六边形架构的一个简单例子。它是一种灵活的体系结构，可以促进分层体系结构和对称等关注点的分离，因为内部应用程序使用端口与外部通信。从现在起，这将是用于构建和解释CQRS和事件来源的基本架构。要更深入地了解这个体系结构，您可以通过一个详细的示例查看附录，解释一些高级主题，如可事务性和其他横切关注点

---

1. 内外分之
2. DIP & DI
3. Port & Adapter
4. PostService 的构造函数依赖PostRepository  合适吗？ PostService 预期应该处理很多业务，都需外部注入存储依赖吗？



