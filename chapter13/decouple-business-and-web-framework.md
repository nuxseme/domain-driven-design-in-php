We have already seen how easy it can be to changing from one persistence strategy to another. However, the persistence is not the only edge from our Hexagon. What about how the user interacts with the application?

Your CTO has set up in the roadmap that your team is moving to Symfony2, so when developing new features in you current ZF1 application, we would like to make the incoming migration easier. That’s tricky, show me your Listing 5.





```php

class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');
        $ideaRepository = new RedisIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute($ideaId, $rating);
        $this->redirect('/idea/'.$ideaId);
    }
}
interface IdeaRepository
{
// ...
}
class RateIdeaUseCase
{
    private $ideaRepository;
    public function __construct(IdeaRepository $ideaRepository)
    {
        $this->ideaRepository = $ideaRepository;
    }
    public function execute($ideaId, $rating)
    {
        try {
            $idea = $this->ideaRepository->find($ideaId);
        } catch(Exception $e) {
            throw new RepositoryNotAvailableException();
        }
        if (!$idea) {
            throw new IdeaDoesNotExistException();
        }
        try {
            $idea->addRating($rating);
            $this->ideaRepository->update($idea);
        } catch(Exception $e) {
            throw new RepositoryNotAvailableException();
        }
        return $idea;
    }
}
```



Let’s review the changes. Our controller is not having any business rules at all. We have pushed all the logic inside a new object called RateIdeaUseCase that encapsulates it. This object is also known as Controller, Interactor or Application Service.

The magic is done by the execute method. All the dependencies such as the RedisIdeaRepository are passed as an argument to the constructor. All the references to an IdeaRepository inside our UseCase are pointing to the interface instead of any concrete implementation.

That’s really cool. If you take a look inside RateIdeaUseCase, there is nothing talking about MySQL or Zend Framework. No references, no instances, no annotations, nothing. It is like your

infrastructure doesn’t mind. It just talks about business logic.

Additionally, we have also tuned the Exceptions we throw. Business processes also have exceptions. NotAvailableRepository and IdeaDoesNotExist are two of them. Based on the one being thrown we can react in different ways in the framework boundary.

Sometimes, the number of parameters that a UseCase receives can be too many. In order to organize them, it’s quite common to build a UseCase request using a Data Transfer Object \(DTO\) to pass them together. Let’s see how you could solve this in Listing 





```php

class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');
        $ideaRepository = new RedisIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );
        $this->redirect('/idea/'.$response->idea->getId());
    }
}
class RateIdeaRequest
{
    public $ideaId;
    public $rating;
    public function __construct($ideaId, $rating)
    {
        $this->ideaId = $ideaId;
        $this->rating = $rating;
    }
}
class RateIdeaResponse
{
    public $idea;
    public function __construct(Idea $idea)
    {
        $this->idea = $idea;
    }
}
class RateIdeaUseCase
{
// ...
    public function execute($request)
    {
        $ideaId = $request->ideaId;
        $rating = $request->rating;
// ...
        return new RateIdeaResponse($idea);
    }
}
```



The main changes here are introducing two new objects, a Request and a Response. They are not mandatory, maybe a UseCase has no request or response. Another important detail is how you build this request. In this case, we are building it getting the parameters from ZF request object.

Ok, but wait, what’s the real benefit? It’s easier to change from one framework to other, or execute our UseCase from another delivery mechanism. Let’s see this point.



