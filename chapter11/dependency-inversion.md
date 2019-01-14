Handling users is not responsibility of the service. As we’ve seen in the repositories chapter, there is a specialised class that deals with User collections, the User repository. This is a dependency from the Application Service to the repository. We don’t want to couple the Application Service with a concrete implementation of the Repository as then, we would be coupling our service with infrastructure details. So we depend on the contract \(interface\) that concrete implementations depend on, the UserRepository.

This dependency will be fulfilled and injected at runtime with a concrete implementation like a

DoctrineUserRepository or even a InMemoryUserRepository on test environment.

Application services could depend on Domain services like GetBadgesByUser too. At runtime the implementation for such a service could be quite elaborated. Imagine a HttpGetBadgesByUser for integrating a bounded context through HTTP protocol.

Depending on abstractions will make our Application Service immune to low-level infrastructure changes.

