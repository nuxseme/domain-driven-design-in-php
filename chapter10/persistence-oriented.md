There are times when collection-oriented repositories do not fit well with our persistence mecha- nism. If you do not have a unit of work, keeping track of Aggregate changes is a difficult task. The only way to persist such changes is by explicitly calling save.

The interface definition for a persistence-oriented repository is similar to how you would define a collection-oriented equivalent.



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





```php
$post =  new  Post(/* ... */);
$postRepository->save($post);

// later ...
$post = $postRepository->postOfId($postId);
$post->editBody(new Body('New body!'));
$postRepository->save($post);
```





Other than this difference, the details are just in the implementation.



