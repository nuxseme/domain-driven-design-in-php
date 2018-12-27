This is the true holder of domain behaviour.

Following on with the example, the Repository interface would be simplified to

```php
interface PostRepository
{

    public function save(Post $post);

    public function byId(PostId $id);

}
```

Now the PostRepository has been freed from all the read concerns except one, the byId which is

responsible for loading the aggregate by its’ ID so that we can operate on it.

And once this is done, all the query methods are also stripped down from the Post model, leaving

it only with command methods. This means we will effectively get rid of all the getter methods and

any other methods exposing information about it. Instead, domain events will be published in order

to be able to trigger write model projections by subscribing to them.

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



All actions that trigger a state change are implemented via domain events. For each domain event

published there is an apply method responsible to reflect the state change



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



