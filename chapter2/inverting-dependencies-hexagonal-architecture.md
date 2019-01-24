Following the essential rule of Layered Architecture, there is a risk to end up implementing domain interfaces that need to make use of infrastructural concerns within the domain model layer.As an example, the PDORepository from the previous example should be placed in the Domain Model if we were following MVC. However, placing infrastructural details right in the middle of our domain violates separation of concerns. This can result in issues, it is hard to avoid violating the essential rules of Layered Architecture, leading to a style of code which can become hard to test if the Domain Layer is aware of technical implementations.

遵循分层体系结构的基本规则，最终实现是有风险的，需要领域模型层中的基础结构关注点的接口有意义。例如，如果我们遵循MVC,前一个例子中的PDORepository应该放在Domain模型。然而，把基础设施的细节放在正中间我们的领域违反了关注点分离。这就会导致问题的出现，是很难避免的分层架构的基本规则，导致一种很难测试的代码风格,领域层耦合技术实现细节

---

1. 每个层处理的边界要定义清晰
2. "任何技术问题多加一个中间层就能解决“，既提供了解决方案，也提供了滥用分层的借口



