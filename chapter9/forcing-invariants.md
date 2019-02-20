Factory Methods in the Aggregate Root are also a good place for invariants.

In a Domain model with a Forum and Article entity, where Article is an Aggregate of Forum, publishing an Article could be something like

聚合根中的工厂方法也是保存不变量的好地方。

在具有论坛和文章实体的域模型中，其中的文章是论坛的集合，发布文章可以类似于这样

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

在与领域专家交谈后，我们得出结论，一个帖子不应该在论坛关闭时发布。这是一个不变量，我们可以在Post创建时直接强制它，以防止不一致的域状态

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



