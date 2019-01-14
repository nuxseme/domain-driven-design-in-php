When there is a unidirectional integration between two Bounded Contexts, where one acts as a provider \(upstream\) and the other as a client of it \(downstream\) we will end up with Customer - Supplier Development Teams.





> Establish a clear customer/supplier relationship between the two teams. In planning sessions, make the downstream team play the customer role to the upstream team. Negotiate and budget tasks for downstream requirements so that everyone understands the commitment and schedule.
>
> Jointly develop automated acceptance tests that will validate the interface expected. Add these tests to the upstream team’s test suite, to be run as part of its’ continuous integration. This testing will free the upstream team to make changes without fear of side effects downstream.
>
> Eric Evans - Domain-Driven Design, Tackling complexity in the heart of software.





Customer  Supplier Development Teams is the most common way of integrating Bounded Contexts. It usually represents a win - win situation when teams work closely.



