Domain Events should be published when the fact they represent happens. For instance, when a new user has been registered, a new UserRegistered event should be published.

Following the newspaper metaphor:



* Modeling a Domain Event is like writing a news article
* Publishing a Domain Event is like printing the article in the paper
* Spreading a Domain Event is like sending the newspaper so everyone can read the article



The recommended approach for publishing DomainEvents is to use a simple Listener-Observer pattern to implement a DomainEventPublisher.



