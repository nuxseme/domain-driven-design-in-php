It’s relatively easy to test entities. Just because they are plain old PHP classes with actions derived of the domain concept they represent. The focus of the test should be the invariants that the entity protects, because probably the behaviour on the entities will be modelled around those invariants.

For the example and for the sake of simplicity, suppose a domain model for a blog is needed. A possible one could be



```php

class Post
{
    private $title;
    private $content; 
    private $status; 
    private $createdAt; 
    private $publishedAt;

    public function construct($aContent, $title)
    {
        $this->setContent($aContent);
        $this->setTitle($title);

        $this->unpublish();
        $this->createdAt(new DateTimeImmutable());
    }

    private function setContent($aContent)
    {
        $this->assertNotEmpty($aContent);

        $this->content = $aContent;
    }

    private function setTitle($aPostTitle)
    {
        $this->assertNotEmpty($aPostTitle);

        $this->title = $aPostTitle;
    }

    private function setStatus(Status $aPostStatus)
    {
        $this->assertIsAValidPostStatus($aPostStatus);

        $this->status = $aPostStatus;
    }

    private function createdAt(DateTimeImmutable $aDate)
    {
        $this->assertIsAValidDate($aDate);
        $this->createdAt = $aDate;
    }

    private function publishedAt(DateTimeImmutable $aDate)
    {
        $this->assertIsAValidDate($aDate);

        $this->publishedAt = $aDate;
    }

    public function publish()
    {
        $this->setStatus( Status::published()
        );

        $this->publishedAt(new DateTimeImmutable());
    }

    public function unpublish()
    {
        $this->setStatus( Status::draft()
        );

        $this->publishedAt = null;
    }

    public function isPublished()
    {
        return $this->status->equalsTo(Status::published());
    }

    public function publicationDate()
    {
        return $this->publishedAt;
    }
}

class Status
{
    const  PUBLISHED  =  10;
    const  DRAFT	=  20;

    private $status;

    public static function published()
    {
        return  new  self(self::PUBLISHED);
    }

    public static function draft()
    {
        return  new  self(self::DRAFT);
    }

    private function construct($aStatus)
    {
        $this->status = $aStatus;
    }

    public function equalsTo(self $aStatus)
    {
        return $this->status === $aStatus->status;
    }
}

```



In order to test this domain model we must ensure the test covers all the Post invariants



```php

class PostTest extends PHPUnit_Framework_TestCase
{
    /** @test */
    public function aNewPostIsNotPublishedByDefault()
    {
        $aPost   =   new Post(
            'A Post Content', 'A Post Title'
        );

        $this->assertFalse(
            $aPost->isPublished()
        );

        $this->assertNull($aPost->publicationDate());
}

    /** @test */
    public function aPostCanBePublishedWithAPublicationDate()
    {
        $aPost   =   new Post(
            'A Post Content', 'A Post Title'
        );

        $aPost->publish();

        $this->assertTrue(
            $aPost->isPublished()
        );

        $this->assertInstanceOf( 'DateTimeImmutable',
            $aPost->publicationDate()
        );
    }
}
```



