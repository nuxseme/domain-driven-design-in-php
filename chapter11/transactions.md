Transactions are detail related with the persistence mechanism. Domain layer should not be aware of this low-level implementation detail. Thinking about beginning, committing or rolling back a transaction at this level is a big smell. This level of detail belongs to the infrastructure layer.

The best way of handling transactions is to not handle them at all \(explicitly\). We could wrap our Application Services with a Decorator implementation for handling the transaction session automatically.



We’ve implemented a solution to this problem in one of our repos, check it out





```php
interface TransactionalSession
{
    /**
     * @return mixed
     */
    public function executeAtomically(callable $operation);
}
```



This contract takes a piece of code and executes it atomically. Depending on your persistence mechanism, you’ll end up with different implementations.

Let’s see how we could do it with Doctrine ORM





```php
class DoctrineSession implements TransactionalSession
{
    private $entityManager;
    public function __construct(EntityManager $entityManager)
    {
        $this->entityManager = $entityManager;
    }
    public function executeAtomically(callable $operation)
    {
        return $this->entityManager->transactional($operation);
    }
}
```



On a sneak peek of how we would make this work on the client for this code.



```php
/** @var EntityManager $em */
$nonTxApplicationService = new SignUpUserService(
    $em->getRepository('BoundedContext\Domain\Model\User\User')
);
$txApplicationService = new TransactionalApplicationService(
    $nonTxApplicationService,
    new DoctrineSession($em)
);
$response = $txApplicationService->execute(
    new SignUpUserRequest(
        'user@example.com',
        'password'
    )
);
```









Now that we have the Doctrine implementation for transactional sessions, it would be great to create a decorator for our Application Services. With this approach we make transactional requests transparent to the Domain.





```php
class TransactionalApplicationService implements ApplicationService
{
    private $session;
    private $service;
    public function __construct(
        ApplicationService $service,
        TransactionalSession $session
    ) {
        $this->session = $session;
        $this->service = $service;
    }
    public function execute($request = null)
    {
        if (empty($this->service)) {
            throw new \LogicException('A use case must be specified');
        }
        $operation = function () use ($request) {
            return $this->service->execute($request);
        };
        return $this->session->executeAtomically($operation);
    }
}
```



A nice side-effect of using Doctrine Session is that it automatically manages the flush method. You don’t need to add the flush inside your Domain or Infrastructure.



