Handling users is not responsibility of the service. As we’ve seen in the repositories chapter, there is a specialised class that deals with User collections, the User repository. This is a dependency from the Application Service to the repository. We don’t want to couple the Application Service with a concrete implementation of the Repository as then, we would be coupling our service with infrastructure details. So we depend on the contract \(interface\) that concrete implementations depend on, the UserRepository.

This dependency will be fulfilled and injected at runtime with a concrete implementation like a

DoctrineUserRepository or even a InMemoryUserRepository on test environment.

Application services could depend on Domain services like GetBadgesByUser too. At runtime the implementation for such a service could be quite elaborated. Imagine a HttpGetBadgesByUser for integrating a bounded context through HTTP protocol.

Depending on abstractions will make our Application Service immune to low-level infrastructure changes.



处理用户不是服务的责任。正如我们在存储库一章中看到的，有一个专门的类处理用户集合，即用户存储库。这是从应用程序服务到存储库的依赖项。我们不希望将应用程序服务与存储库的具体实现耦合起来，因为那样的话，我们将把服务与基础结构细节耦合起来。因此，我们依赖于具体实现所依赖的契约\(接口\)，即UserRepository。

该依赖关系将在运行时通过一个具体的实现\(如a\)实现并注入

DoctrineUserRepository，甚至测试环境上的InMemoryUserRepository。

应用程序服务也可以依赖于像GetBadgesByUser这样的域服务。在运行时，这种服务的实现可以非常详细。想象一个HttpGetBadgesByUser通过HTTP协议集成了一个有边界的上下文。

依赖于抽象将使我们的应用程序服务不受底层基础结构更改的影响。

