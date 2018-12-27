Following on with the blog example application, the first concept we need is a Port where the outside

world could talk to the application. For this case, we will use an HTTP Port and its corresponding

Adapter. The outside will use the port to send messages to the application. The example was using

a database to store the whole collection of blog posts so, in order to allow the application retrieve

blog posts from the database, a Port is needed



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



```
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

