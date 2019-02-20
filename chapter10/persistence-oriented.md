There are times when collection-oriented repositories do not fit well with our persistence mecha- nism. If you do not have a unit of work, keeping track of Aggregate changes is a difficult task. The only way to persist such changes is by explicitly calling save.

The interface definition for a persistence-oriented repository is similar to how you would define a collection-oriented equivalent.

有时候，面向集合的存储库并不适合我们的持久性机制。如果您没有工作单元，那么跟踪聚合更改是一项困难的任务。保存这些更改的惟一方法是显式调用save。

面向持久性存储库的接口定义类似于您将如何定义面向集合的对等物。

```php
interface PostRepository
{
    public function nextIdentity();
    public function postOfId(PostId $anId);
    public function save(Post $aPost);
    public function saveAll(array $posts);
    public function remove(Post $aPost);
    public function removeAll(array $posts);
}
```

In this case we now have save and saveAll methods. They provide similar functionality to the previous add and addAll methods, however, the important difference is how the client uses them. Within a collection-oriented style, you use the add methods just once, when the Aggregate is created. In a persistence-oriented style, you will not only use the save action after creating the Aggregate, but also when they are modified.

在本例中，我们现在有save和saveAll方法。它们提供了与前面的add和addAll方法类似的功能，但是，重要的区别是客户机如何使用它们。在面向集合的样式中，只在创建聚合时使用一次add方法。在面向持久性的样式中，不仅在创建聚合之后使用save操作，而且在修改聚合时也使用save操作。

```php
$post =  new  Post(/* ... */);
$postRepository->save($post);

// later ...
$post = $postRepository->postOfId($postId);
$post->editBody(new Body('New body!'));
$postRepository->save($post);
```

Other than this difference, the details are just in the implementation.

除了这个区别之外，细节只是在实现中。

