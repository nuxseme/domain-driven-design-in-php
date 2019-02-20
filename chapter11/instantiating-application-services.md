Instantiating just your Application Service is easy. Building the dependency tree might be tricky depending on how complicated are the dependencies to build. For such a purpose, most frameworks come with a Dependency Injection container. Without one, you’ll end up with something like the following code somewhere in your controller.

实例化应用程序服务很容易。根据要构建的依赖项有多复杂，构建依赖项树可能比较棘手。出于这个目的，大多数框架都带有依赖注入容器。如果没有，您将在控制器中得到如下代码。

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

我们决定使用UserRepository复述,²的实现。还有所有的细节我们需要建立存储库,如Predis³配置。这不仅效率低下，还会在控制器之间传播重复。

您可以将构造逻辑重构到工厂中，也可以使用依赖注入容器。大多数现代框架都是这样的。

> Is it bad to use a Dependency Injection Container?
>
> Not at all. Dependency Injection Containers are just a tool. They help abstracting away the complexities of building your dependencies. They come very handy for building infrastructure artifacts. Symfony offers a complete solution⁴.
>
> Take into account that passing the entire container as a whole to one of services is a bad practice. That would be like coupling the entire context of your application with the domain. If you fetch an object for a specific service, do it. Don’t make that service aware of the entire context.
>
> 使用依赖注入容器不好吗?
>
>
>
> 不客气。依赖注入容器只是一种工具。它们有助于抽象构建依赖项的复杂性。它们对于构建基础设施构件非常方便。⁴Symfony提供了一个完整的解决方案。
>
>
>
> 考虑到将整个容器作为一个整体传递给一个服务是一种不好的做法。这就像是将应用程序的整个上下文与域耦合起来。如果您为特定的服务获取对象，那么就这样做。不要让服务知道整个上下文。

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

如您所见，$app被用作服务容器。我们注册所有需要的组件及其依赖项。sign\_up\_user\_application\_service依赖于上面的定义。更改user\_repository的实现很容易，因为返回的是其他内容\(MySQL、MongoDB等\)，根本不需要更改服务代码。

类似于Symfony应用程序

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

现在您已经在Symfony服务容器中定义了应用程序服务，在任何上下文中获取服务都非常简单。所有的交付机制共享相同的定义、Web控制器、REST控制器甚至控制台命令。该服务可用于实现ContainerAware接口的任何类。得到这项服务和打电话一样容易

```php
$this->get('sign_up_user_application_service').
```

To sum up, how do you build your services \(ad-hoc, using Service Container, Factories, etc.\) does not matter. It’s important to keep it out to the Infrastructure boundary.

总之，如何构建服务\(特别的，使用服务容器、工厂等\)并不重要。将其限制在基础设施边界之外是很重要的。

