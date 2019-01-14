Are we forgetting something? “the author should be notified by email”, yeah! That’s true. Let’s see in Listing 16 how we have updated the UseCase for doing the job.





```php


class RateIdeaUseCase
{
    private $ideaRepository;
    private $authorNotifier;
    public function __construct(
        IdeaRepository $ideaRepository,
        AuthorNotifier $authorNotifier
    )
    {
        $this->ideaRepository = $ideaRepository;
        $this->authorNotifier = $authorNotifier;
    }
    public function execute(RateIdeaRequest $request)
    {
        $ideaId = $request->ideaId;
        $rating = $request->rating;
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
        try {
            $this->authorNotifier->notify(
                $idea->getAuthor()
            );
        } catch(Exception $e) {
            throw new NotificationNotSentException();
        }
        return $idea;
    }
}
```

As you realize, we have added a new parameter for passing AuthorNotifier Service that will send the email to the author. This is the port in the “Ports and Adapters” naming. We have also updated the business rules in the execute method.

Repositories are not the only objects that may access your infrastructure and should be decoupled using interfaces or abstract classes. Domain Services can too. When there is a behavior not clearly owned by just one Entity in your domain, you should create a Domain Service. A typical pattern is to write an abstract Domain Service that has some concrete implementation and some other abstract methods that the adapter will implement.

As an exercise, define the implementation details for the AuthorNotifier abstract service. Options are SwiftMailer or just plain mail calls. It’s up to you.



