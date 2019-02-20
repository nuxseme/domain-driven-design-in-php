Transactions are detail related with the persistence mechanism. Domain layer should not be aware of this low-level implementation detail. Thinking about beginning, committing or rolling back a transaction at this level is a big smell. This level of detail belongs to the infrastructure layer.

The best way of handling transactions is to not handle them at all \(explicitly\). We could wrap our Application Services with a Decorator implementation for handling the transaction session automatically.

We’ve implemented a solution to this problem in one of our repos, check it out

事务是与持久性机制相关的细节。域层不应该知道这个低级实现细节。考虑在这个级别开始、提交或回滚事务是一件很麻烦的事情。此详细级别属于基础结构层。

处理事务的最佳方法是根本不处理它们\(显式地\)。我们可以使用装饰器实现包装应用程序服务，以便自动处理事务会话。

我们已经在我们的一个repos中实现了这个问题的解决方案，请查看

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

这个契约接受一段代码并以原子方式执行它。根据您的持久性机制，您将得到不同的实现。

让我们看看如何运用 Doctrine ORM 

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

让我们看一看如何在客户机上实现这段代码。

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

既然我们已经有了事务会话的Doctrine实现，那么为我们的应用程序服务创建一个装饰器就很好了。通过这种方法，我们使事务请求对域透明。

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

使用Doctrine会话的一个很好的副作用是它自动管理刷新方法。您不需要在域或基础结构中添加刷新。

