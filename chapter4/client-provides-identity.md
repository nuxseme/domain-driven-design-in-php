Sometimes, dealing with certain domains, the identities come naturally with the client consuming the domain model. Probably this is the ideal case, because the identity can be modelled quite easy

```php
namespace Ddd\Catalog\Domain\Model\Book;

class ISBN
{
    private $isbn;

    private function construct($anIsbn)
    {
        $this->setIsbn($anIsbn);
    }

    private function setIsbn($anIsbn)
    {
        $this->assertIsbnIsValid($anIsbn, 'The ISBN is invalid.');

        $this->isbn = $anIsbn;
    }

    public static function create($anIsbn)
    {
        return new static($anIsbn);
    }

    private function assertIsbnIsValid($anIsbn, $string)
    {
// ... Validates an ISBN code
    }
}

class Book
{
    private $isbn;
    private $title;

    public function construct(ISBN $anIsbn, $aTitle)
    {
        $this->isbn = $anIsbn;
        $this->title = $aTitle;
    }
}



$book = new Book( ISBN::create('...'),
    'Domain-Driven Design with PHP by Examples'
);
```



