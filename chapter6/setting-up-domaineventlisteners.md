Where is the best place to set up the subscribers to the DomainEventPublisher? It depends. For global subscribers that affect all the request, probably when building your DomainEventPublisher. If some subscribers just affect a specific Application Service, when building the Application Service. Let’s see an example using Silex.

为DomainEventPublisher设置订阅者的最佳位置在哪里?视情况而定。对于影响所有请求的全局订阅者，可能是在构建您的DomainEventPublisher时。如果某些订阅者只影响特定的应用程序服务，则在构建应用程序服务时。让我们看一个使用Silex的示例。

In Silex⁷, the best way to register a Domain Event Publisher that will persist all Domain Events is using an Application Middleware⁸. A before application middleware allows you to tweak the Request before the controller is executed. It’s the right place to subscribe the listener responsible for persisting those events to the database that will be send to RabbitMQ later.

在Silex⁷,注册一个域事件发布者最好的方法将持续使用应用程序中间件⁸所有域事件。before应用程序中间件允许您在控制器执行之前调整请求。它是订阅负责将这些事件持久化到数据库\(稍后将发送到RabbitMQ\)的侦听器的正确位置。

```php
// ...
$app['em'] = $app->share(function() {
    return (new EntityManagerFactory())->build();
});

$app['event_repository'] = $app->share(function($app) {
    return $app['em']->getRepository('Ddd\Domain\Model\Event\StoredEvent');
});

$app['event_publisher'] = $app->share(function($app) { return DomainEventPublisher::instance();
});

$app->before(function (Symfony\Component\HttpFoundation\Request $request) use ($app) {
    $app['event_publisher']->subscribe( new PersistDomainEventSubscriber(
            $app['event_repository']
        )
    );
});
```

With this setup, each time an Aggregate will publish a DomainEvent, it will get persisted into the database. Mission accomplished.

通过这种设置，每当聚合发布一个DomainEvent时，它将被持久化到数据库中。任务完成

