A common implementation for the criterion object is the Specification Pattern. A specification is just a simple predicate that takes a domain object and returns a boolean. Given a domain object, it will return true if it specifies the specification and false otherwise.



```php
interface PostSpecification
{
    /**
    * @return boolean
    */
    public function specifies(Post $aPost);
}
```



We just need to add a query method to our repository.



```php
interface PostRepository
{
    public function query($specification);
}
```



