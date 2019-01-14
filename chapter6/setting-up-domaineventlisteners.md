Where is the best place to set up the subscribers to the DomainEventPublisher? It depends. For global subscribers that affect all the request, probably when building your DomainEventPublisher. If some subscribers just affect a specific Application Service, when building the Application Service. Let’s see an example using Silex.

In Silex⁷, the best way to register a Domain Event Publisher that will persist all Domain Events is using an Application Middleware⁸. A before application middleware allows you to tweak the Request before the controller is executed. It’s the right place to subscribe the listener responsible for persisting those events to the database that will be send to RabbitMQ later.

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

