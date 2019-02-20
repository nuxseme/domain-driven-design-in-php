The Domain Model is not the place to manage transactions. The operations applied over the Domain

Model should be agnostic of the persistence mechanism. A common approach to solving this problem

is placing a Facade¹⁰ in the Application Layer, grouping related Use Cases together. When a method

of the Facade is invoked from the User Interface Layer, the business method begins a transaction.

Once complete, the Facade ends the interaction by committing the transaction. If anything goes

wrong, the transaction is rolled back.

域模型不是管理事务的地方。在域上应用的操作模型应该与持久性机制无关。一种解决这个问题的通用方法在应用层放置Facade，将相关用例分组在一起。当一个方法从用户界面层调用Facade的，业务方法开始一个事务。一旦完成，Facade通过提交事务来结束交互。如果任何事情发生错误，事务回滚。

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

facades引入的问题是，我们必须在和上重复相同的样板代码结束了。如果我们统一执行用例的方式，我们可以使用装饰器模式

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

我们不希望应用程序层与具体的事务过程耦合，因此我们可以为它创建一个简单的接口

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

可以使任何应用程序服务具有事务性的已实现的装饰器如下所示

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

接下来，我们可以创建一个Doctrine事务会话实现

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

现在我们有了在事务中执行用例的一切

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



