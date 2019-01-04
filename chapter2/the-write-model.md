This is the true holder of domain behaviour. Following on with the example, the Repository interface would be simplified to

这才是领域行为的真正持有者。在接下来的示例中，存储库接口将简化为

```php
interface PostRepository
{

    public function save(Post $post);

    public function byId(PostId $id);

}
```

Now the PostRepository has been freed from all the read concerns except one, the byId which is responsible for loading the aggregate by its’ ID so that we can operate on it. And once this is done, all the query methods are also stripped down from the Post model, leaving it only with command methods. This means we will effectively get rid of all the getter methods and any other methods exposing information about it. Instead, domain events will be published in order to be able to trigger write model projections by subscribing to them.

现在post - strepository已经从所有读问题中解放出来了，除了一个，byId，它负责以' ID '装载聚合，以便我们可以对其进行操作。一旦完成，所有的查询方法也将从Post模型中剥离出来，只剩下命令方法。这意味着我们将有效地摆脱所有getter方法和任何其他暴露有关它的信息的方法。相反，将发布域事件，以便通过订阅它们来触发写模型投影



```php
class AggregateRoot
{

    private $recordedEvents = [];

    protected function recordApplyAndPublishThat(DomainEvent $domainEvent)
    {

        $this->recordThat($domainEvent);

        $this->applyThat($domainEvent);

        $this->publishThat($domainEvent);

    }

    protected function recordThat(DomainEvent $domainEvent)
    {

        $this->recordedEvents[] = $domainEvent;

    }

    protected function applyThat(DomainEvent $domainEvent)
    {

        $modifier = 'apply' . get_class($domainEvent);

        $this->$modifier($domainEvent);

    }

    protected function publishThat(DomainEvent $domainEvent)
    {

        DomainEventPublisher::getInstance()->publish($domainEvent);

    }

    public function recordedEvents()
    {

        return $this->recordedEvents;

    }

    public function clearEvents()
    {

        $this->recordedEvents = [];

    }
}
```

```php
class Post extends AggregateRoot
{

    private $id;

    private $title;

    private $content;

    private $published = false;

    private $categories;

    private function __construct(PostId $id)
    {

        $this->id = $id;

        $this->categories = new Collection();

    }

    public static function writeNewFrom($title, $content)
    {

        $post = new Post(PostId::create());

        $post->recordApplyAndPublishThat(

            new PostWasCreated(PostId::generate(), $title, $content)

        );

    }

    public function publish()
    {

        $this->recordApplyAndPublishThat(

            new PostWasPublished($this->id)

        );

    }

    public function categorizeIn(CategoryId $categoryId)
    {

        $this->recordApplyAndPublishThat(

            new PostWasCategorized($this->id, $categoryId)

        );

    }

    public function changeContentFor($newContent)
    {

        $this->recordApplyAndPublishThat(

            new PostContentWasChanged($this->id, $newContent)

        );

    }

    public function changeTitleFor($newTitle)
    {

        $this->recordApplyAndPublishThat(

            new PostTitleWasChanged($this->id, $newTitle)

        );
    }
}
```

All actions that trigger a state change are implemented via domain events. For each domain event published there is an apply method responsible to reflect the state change

触发状态更改的所有操作都是通过域事件实现的。对于发布的每个域事件，都有一个apply方法负责反映状态变化

```php
class Post extends AggregateRoot
{
// ...
    protected function applyPostWasCreated(PostWasCreated $event)
    {
        $this->id      = $event->id();
        $this->title   = $event->title();
        $this->content = $event->content();
    }

    protected function applyPostWasPublished(PostWasPublished $event)
    {
        $this->published = true;
    }

    protected function applyPostWasCategorized(PostWasCategorized $event)
    {
        $this->categories->add($event->categoryId());
    }

    protected function applyPostContentWasChanged(PostContentWasChanged $event)
    {
        $this->content = $event->content();
    }

    protected function applyPostTitleWasChanged(PostTitleWasChanged $event)
    {
        $this->title = $event->title();
    }
}
```

---

1. 模型行为解耦  ...



