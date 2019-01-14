If you want to achieve 100% coverage for your whole application you will also have to test your infrastructure. Before doing that, you need to know that those unit tests will be more coupled to your implementation than the business ones. That means that the probability to be broken with implementation details changes is higher. So it’s a trade-off you will have to consider.

So, if you want to continue, we need to do some modifications. We need to decouple even more. Let’s see the code in Listing



```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');
        $useCase = new RateIdeaUseCase(
            new RedisIdeaRepository(
                new \Predis\Client()
            )
        );
        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );
        $this->redirect('/idea/'.$response->idea->getId());
    }
}
class RedisIdeaRepository implements IdeaRepository
{
    private $client;
    public function __construct($client)
    {
        $this->client = $client;
    }
    // ...
    public function find($id)
    {
        $idea = $this->client->get($this->getKey($id));
        if (!$idea) {
            return null;
        }
        return $idea;
    }
}
```







If we want to 100% unit test RedisIdeaRepository we need to be able to pass the Predis\Client as a parameter to the repository without specifying TypeHinting so we can pass a mock to force the code flow necessary to cover all the cases.

This forces us to update the Controller to build the Redis connection, pass it to the repository and pass the result to the UseCase.

Now, it’s all about creating mocks, test cases and having fun doing asserts.



