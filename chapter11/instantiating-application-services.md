Instantiating just your Application Service is easy. Building the dependency tree might be tricky depending on how complicated are the dependencies to build. For such a purpose, most frameworks come with a Dependency Injection container. Without one, you’ll end up with something like the following code somewhere in your controller.

```php
$redisClient = new Predis\Client([
    'scheme' => 'tcp',
    'host' => '10.0.0.1',
    'port' => 6379
]);
$userRepository = new RedisUserRepository($redisClient);
$signUp = new SignUpUserService($userRepository);
$signUp->execute(new SignUpUserRequest(
    'user@example.com',
    'password'
));
```

We decided to use the Redis² implementation for the UserRepository. There are all the details we need to build that repository right there, like Predis³ configuration. This is not only inefficient it also spreads duplication across controllers.

You could refactor the construction logic into a Factory or you could use a Dependency Injection Container. Most of modern frameworks come with it.





> Is it bad to use a Dependency Injection Container?
>
> Not at all. Dependency Injection Containers are just a tool. They help abstracting away the complexities of building your dependencies. They come very handy for building infrastructure artifacts. Symfony offers a complete solution⁴.



> Take into account that passing the entire container as a whole to one of services is a bad practice. That would be like coupling the entire context of your application with the domain. If you fetch an object for a specific service, do it. Don’t make that service aware of the entire context.





Let’s see how would we do it with Silex

```php

// ...
$app = new \Silex\Application();
$app['redis_parameters'] = [
    'scheme' => 'tcp',
    'host' => '127.0.0.1',
    'port' => 6379
];
$app['redis'] = $app->share(function ($app) {
    return new Predis\Client($app['redis_parameters']);
});
$app['user_repository'] = $app->share(function ($app) {
    return new RedisUserRepository(
        $app['redis']
    );
});
$app['sign_up_user_application_service'] = $app->share(function ($app) {
    return new SignUpUserService(
        $app['user_repository']
    );
});
//...
$app->match('/signup', function (Request $request) use ($app) {
//...
    $app['sign_up_user_application_service']->execute(
        new SignUpUserRequest(
            $request->get('email'),
            $request->get('password')
        )
    );
//...
});
}
```





As you can see, $app is used as the Service Container. We register all the components needed and their dependencies. sign\_up\_user\_application\_service depends on the definitions made above. Changing the implementation for the user\_repository is a easy as returning something else \(MySQL, MongoDB, etc.\), we don’t need to change the service code at all.

The equivalent for a Symfony application



```php
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://symfony.com/schema/dic/services
http://symfony.com/schema/dic/services/services-1.0.xsd">
<services>
<service
id="sign_up_user_application_service"
class="SignUpUserService">
<argument type="service" id="user_repository" />
</service>
<service
id="user_repository"
class="RedisUserRepository">
<argument type="service">
<service class="Predis\Client" />
</argument>
</service>
</services>
</container>
```

Now that you have the definition of your Application Service in the Symfony Service container, getting the service at any context is pretty straightforward. All delivery mechanisms share the same definition, Web Controllers, REST controllers or even Console Commands. The Service is available on any class implementing the ContainerAware interface. Getting the service is as easy as calling

```php
$this->get('sign_up_user_application_service').
```

To sum up, how do you build your services \(ad-hoc, using Service Container, Factories, etc.\) does not matter. It’s important to keep it out to the Infrastructure boundary.

