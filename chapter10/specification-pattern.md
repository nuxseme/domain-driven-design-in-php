A common implementation for the criterion object is the Specification Pattern. A specification is just a simple predicate that takes a domain object and returns a boolean. Given a domain object, it will return true if it specifies the specification and false otherwise.

规范对象的一个常见实现是规范模式。规范只是一个简单的谓词，它接受域对象并返回布尔值。给定一个域对象，如果它指定了规范，则返回true，否则返回false。

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

我们只需要向存储库中添加一个查询方法。

```php
interface PostRepository
{
    public function query($specification);
}
```



