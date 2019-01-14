Test Data Builders are just normal Builders with default values used exclusively in your test suites so you don’t have to specify irrelevant parameters on specific test cases.



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



