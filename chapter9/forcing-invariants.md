Factory Methods in the Aggregate Root are also a good place for invariants.

In a Domain model with a Forum and Article entity, where Article is an Aggregate of Forum, publishing an Article could be something like



```php
class Forum
{
// ...

    public function publishPost(PostId $postId, $content)
    {
        $post = new Post($this->id, $postId, $content);

        DomainEventPublisher::instance()->publish(
            new PostPublished($postId)
        );

        return $post;
    }
}
```



After talking with a Domain Expert we came to the conclusion that a Post should not be published when the Forum is closed. This is an invariant and we could force it directly on Post creation preventing an inconsistent Domain state



```php
class Forum
{
// ...

    public function publishPost(PostId $postId, $content)
    {
        if ($this->isClosed()) {
            throw new ForumClosedException();
        }
        $post = new Post($this->id, $postId, $content); DomainEventPublisher::instance()->publish(
        new PostPublished($postId)
    );

        return $post;
    }
}
```



