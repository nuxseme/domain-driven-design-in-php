Domain Event listeners have to be configured before the Application Service gets executed or nobody will be noticed. There are situations where you’ll have to be explicitly and configure the listener before executing the Application Service.



```php
//...
$subscriber = new SpySubscriber();
DomainEventPublisher::instance()->subscribe($subscriber);
$applicationService = //...
$applicationService->execute(...);
```

Most of the time this will be done by configuring the Dependency Injection Container.



