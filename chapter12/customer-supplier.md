When there is a unidirectional integration between two Bounded Contexts, where one acts as a provider \(upstream\) and the other as a client of it \(downstream\) we will end up with Customer - Supplier Development Teams.

> Establish a clear customer/supplier relationship between the two teams. In planning sessions, make the downstream team play the customer role to the upstream team. Negotiate and budget tasks for downstream requirements so that everyone understands the commitment and schedule.
>
> Jointly develop automated acceptance tests that will validate the interface expected. Add these tests to the upstream team’s test suite, to be run as part of its’ continuous integration. This testing will free the upstream team to make changes without fear of side effects downstream.
>
> Eric Evans - Domain-Driven Design, Tackling complexity in the heart of software.

Customer  Supplier Development Teams is the most common way of integrating Bounded Contexts. It usually represents a win - win situation when teams work closely.



当两个边界上下文之间存在单向集成时，其中一个充当提供者\(上游\)，另一个充当it客户\(下游\)，我们最终将拥有客户-供应商开发团队。

> 在两个团队之间建立清晰的客户/供应商关系。在规划会议时，让下游团队对上游团队扮演客户角色。为下游需求谈判和预算任务，使每个人都了解承诺和进度。
>
>
>
> 共同开发自动化的验收测试，以验证预期的接口。将这些测试添加到上游团队的测试套件中，作为其“持续集成”的一部分运行。这个测试将使上游团队能够自由地进行更改，而不用担心下游的副作用。
>
>
>
> 领域驱动的设计，解决软件核心的复杂性。

客户供应商开发团队是集成有限上下文的最常见方法。当团队紧密合作时，它通常代表一个双赢的局面。

