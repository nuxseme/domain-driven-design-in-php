During the sprint and after talking to some mates, you realize that using a NoSQL strategy could

improve the performance of your feature. Redis is one of your best friends. Go for it and show me

your Listing 4.



```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');
        $ideaRepository = new RedisIdeaRepository();
        $idea = $ideaRepository->find($ideaId);
        if (!$idea) {
            throw new Exception('Idea does not exist');
        }
        $idea->addRating($rating);
        $ideaRepository->update($idea);
        $this->redirect('/idea/'.$ideaId);
    }
}
interface IdeaRepository
{
// ...
}
class RedisIdeaRepository implements IdeaRepository
{
    private $client;
    public function __construct()
    {
        $this->client = new \Predis\Client();
    }
    public function find($id)
    {
        $idea = $this->client->get($this->getKey($id));
        if (!$idea) {
            return null;
        }
        return unserialize($idea);
    }
    public function update(Idea $idea)
    {
        $this->client->set(
            $this->getKey($idea->getId()),
            serialize($idea)
        );
    }
    private function getKey($id)
    {
        return 'idea:' . $id;
    }
}
```



Easy again. You’ve created a RedisIdeaRepository that implements IdeaRepository interface and

we have decided to use Predis as a connection manager. Code looks smaller, easier and faster. But

what about the controller? It remains the same, we have just changed which repository to use, but

it was just one line of code.

As an exercise for the reader, try to create the IdeaRepository for SQLite, a file or an in-memory

implementation using arrays. Extra points if you think about how ORM Repositories fit with Domain

Repositories and how ORM @annotations affect this architecture.

