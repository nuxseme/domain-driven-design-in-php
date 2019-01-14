The Domain Model is not the place to manage transactions. The operations applied over the Domain

Model should be agnostic of the persistence mechanism. A common approach to solving this problem

is placing a Facade¹⁰ in the Application Layer, grouping related Use Cases together. When a method

of the Facade is invoked from the User Interface Layer, the business method begins a transaction.

Once complete, the Facade ends the interaction by committing the transaction. If anything goes

wrong, the transaction is rolled back.



```php
use Doctrine\ORM\EntityManager;
class SomeApplicationServiceFacade
{
    private $em;
    public function __construct(EntityManager $em)
    {
        $this->em = $em;
    }
    public function doSomeUseCaseTask()
    {
        try {
            $this->em->getConnection()->beginTransaction();
// Use domain model
            $this->em->getConnection()->commit();
        } catch(Exception $e) {
            $this->em->getConnection()->rollback();
            throw $e;
        }
    }
}
```



The problem introduced with facades is that we have to repeat the same boilerplate code over and

over. If we unify the way we execute use cases, we could wrap them in a transaction using the

Decorator Pattern

```php
interface ApplicationService
{
    /**
     * @param $request
     * @return mixed
     */
    public function execute($request = null);
}

class SomeApplicationService implements ApplicationService
{
    public function execute($request = null)
    {
// do something
    }
}
```



We do not want to couple our Application Layer with the concrete transactional procedure, so instead we can create a simple interface for it

```php

interface TransactionalSession
{
    /**
     * @param callable $operation
     * @return mixed
     */
    public function executeAtomically(callable $operation);
}
```

The implemented decorator that can make any application service transactional is as easy as the following



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
        $operation = function () use ($request) {
            return $this->service->execute($request);
        };
        return $this->session->executeAtomically($operation->bindTo($this));
    }
}
```



Following this, we could alternatively create a Doctrine transactional session implementation



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

Now we have everything to execute our Use Cases within a transaction



```php
$useCase = new TransactionalApplicationService(
    new SomeApplicationService(
// ...
    ),
    new DoctrineSession(
// ...
    )
);
$response = $useCase->execute();
```



