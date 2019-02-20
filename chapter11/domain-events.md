Domain Event listeners have to be configured before the Application Service gets executed or nobody will be noticed. There are situations where you’ll have to be explicitly and configure the listener before executing the Application Service.

必须在执行应用程序服务之前配置域事件侦听器，否则将不会有人注意到。在某些情况下，您必须在执行应用程序服务之前显式地配置侦听器。

```php
//...
$subscriber = new SpySubscriber();
DomainEventPublisher::instance()->subscribe($subscriber);
$applicationService = //...
$applicationService->execute(...);
```

Most of the time this will be done by configuring the Dependency Injection Container.

大多数情况下，这将通过配置依赖项注入容器来完成。

