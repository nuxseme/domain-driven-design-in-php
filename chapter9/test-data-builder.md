Test Data Builders are just normal Builders with default values used exclusively in your test suites so you don’t have to specify irrelevant parameters on specific test cases.

测试数据构建器只是普通的构建器，其默认值只在您的测试套件中使用，因此您不必在特定的测试用例上指定无关的参数。

```php
class AuthorBuilder
{
    private $username;
    private $email;
    private $fullName;

    private function construct()
    {
        $this->username = new Username('johndoe');
        $this->email = new Email('john@doe.com');
        $this->fullName = new FullName('John', 'Doe');
    }

    public static function anAuthor()
    {
        return new self();
    }

    public function withFullName(FullName $aFullName)
    {
        $this->fullName = $aFullName;

        return $this;
    }

    public function withUsername(Username $aUsername)
    {
        $this->username  =  $aUsername;

        return $this;
    }

    public function withEmail(Email $anEmail)
    {
        $this->email = $anEmail;

        return $this;
    }

    public function build()
    {
        return  new  Author($this->username,  $this->fullName,  $this->email);
    }
}



class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $author = AuthorBuilder::anAuthor()
            ->withEmail(new Email('other@email.com'))
            ->build();

        //do something with author
    }
}
```

We could even combine Test Data Builders to build more complicated aggregates like a Post

我们甚至可以组合测试数据构建器来构建更复杂的聚合，比如Post

```php
class Post
{
    private $id; private $author; private $body; private $createdAt;

    public function construct( PostId $anId,
        Author $anAuthor, Body $aBody)
    {
        $this->id = $anId;
        $this->author = $anAuthor;
        $this->body  =  $aBody;
        $this->createdAt = new DateTime();
    }
}
```

And the its respective Test Data Builder. We could reuse the AuthorBuilder for building a default Author

以及它各自的测试数据构建器。我们可以重用AuthorBuilder来构建默认作者

```php
class PostBuilder
{
    private $postId; 
    private $author; 
    private $body;

    private function construct()
    {
        $this->postId = new PostId();
        $this->author = AuthorBuilder::anAuthor()->build();
        $this->body = new Body('Post body');
    }

    public static function aPost()
    {
        return new self();
    }

    public function withAuthor(Author $anAuthor)
    {
        $this->author = $anAuthor;

        return $this;
    }

    public function withPostId(PostId $aPostId)
    {
        $this->postId = $aPostId;

        return $this;
    }

    public function withBody(Body $body)
    {
        $this->body = $body;

        return $this;
    }

    public function build()
    {
        return new Post($this->postId, $this->author, $this->body);
    }
}
```

This solution is now flexible enough to adapt our fixtures to any kind of flow in the system under test, including the possibility of building inner entities.

这个解决方案现在已经足够灵活，可以使我们的夹具适应测试系统中的任何类型的流，包括构建内部实体的可能性。

```php
class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $post = PostBuilder::aPost()
            ->withAuthor(AuthorBuilder::anAuthor()
                ->withUsername(new Username('other'))
                ->build())
            ->withBody(new Body('Another body'))
            ->build();

        //do something with the post
    }
}
```



