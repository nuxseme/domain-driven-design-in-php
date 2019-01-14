In Domain-Driven Design, transactions are handled at the Application Service level. You are not going to find any beginTransaction or similar anywhere in your Domain code. All the operations performed during the execution of the Application Service are going to be run atomically against your database.

In PHP, we have an elegant solution in order to make your Application Services be executed in a transactional scope without worrying about transactions in your Domain code.



You can possibly think that persisting a new user is just going to run a single insert on a table, but what if we have more than one table storing different information about the users?

```php
interface Service
{
    public function execute($request);
}

class TransactionalService
{
    private $session;
    private $service;

    public function construct(Service $service, TransactionalSession $session)
    {
        $this->session   = $session;
        $this->service  =  $service;
    }

    public function execute($request)
    {
        if (empty($this->service)) {
            throw new \LogicException('A use case must be specified');
        }

        $operation = function () use ($request) {
            return $this->service->execute($request);
        };

        return $this->session->executeAtomically($operation->bindTo($this));
    }
}

interface TransactionalSession
{
    /**
     * @return mixed
     */
    public function executeAtomically(callable $operation);
}
```



