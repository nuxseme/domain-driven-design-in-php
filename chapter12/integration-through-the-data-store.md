One of the most commonly used techniques to integrate different parts of an application has always been to share the same data store, along with the same code base. This is usually known as a monolithic application, often ending up with a single data-store that hosts the data related to all the concerns within the application.

Consider an e-commerce application, a shared data-store would contain all concerns \(eg: tables within a relational database\) surrounding the catalog, billing, inventory, and so on. There is nothing bad with this approach per se, for example in small linear applications were the complexity is not too high. However, within complex domains, some issues can arise. If you share data across many tables touching multiple application concerns, transactions will have a big impact on performance.

Another less technical problem that could develop will be in-regard to the Ubiquitous Language. The main advantage to the seperation of Bounded Contexts is having a single Ubiquitous Language for each one. In doing so, models will be separated into their own contexts. Mixing all models together within the same context can lead to ambiguity and confusion.

Going back to the e-commerce system, imagine we want to introduce the concept of a t-shirt. Within the catalogue context, a t-shirt would be a product with properties like color, size, material and maybe some fancy pictures. In the inventory system however, we do not really wish to concern



ourselves with these. A product here has a different meaning, were we care about different properties like weight, location in the warehouse or dimensions. Mixing both contexts together will tangle concepts and will complicate the design. In DDD terms, mixing concepts in this manner is what is called a Shared Kernel.



> Shared Kernel
>
> Designate some subset of the domain model that the teams agree to share. Of course this
>
> includes, along with this subset of the model, the subset of code or of the database design associated with that part of the model. This explicitly shared stuff has special status, and shouldn’t be changed without consultation with the other team. Integrate a functional system frequently, but somewhat less often than the pace of CONTINUOUS INTEGRATION within the teams. At these integrations, run the tests of both teams.
>
> Eric Evans - Domain-Driven Design, Tackling complexity in the heart of software. Chapter 14 - Shared Kernel



We do not recommend using a Shared Kernel. As multiple teams can collide within the development of it, ending up having maintenance issues and becoming a friction point. Changes in the Shared Kernel should be agreed upon beforehand, between all parties involved. Conceptually it has other problems, such as people seeing it as a bag to place ‘stuff’ that does not belong anywhere else, growing indefinitely.

A better way of dealing with the ever growing monolithic complexity is to break it up in different autonomous pieces. Such as communicating through REST, RPC or messaging systems. This requires drawing clear boundaries, with each context likely ending up with their own infrastructure – data stores, servers, messaging middleware, etc. – and even its own team. As you may foresee, this could lead to some degree of duplication. That is a trade-off that we are willing to make in order to reduce complexity. These autonomous pieces receive the name of Bounded Contexts.





