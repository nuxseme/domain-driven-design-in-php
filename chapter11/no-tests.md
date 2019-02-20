Application requests are data structures not objects. Unit testing data structures is like testing getters and setters. There is no behaviour to test so there is not that much value trying to unit testing them. This structures will be covered as a side-effect of more elaborated tests like Integration or Acceptance tests.

An alternative to request objects are Commands. We could design an Service with multiple Application methods. Each one of them with the parameters you’d put inside the Request. It’s okay for simple applications. No worries, we’ll come back to this topic later.

应用程序请求是数据结构而不是对象。单元测试数据结构就像测试getter和setter。没有要测试的行为，所以对它们进行单元测试没有多大价值。这种结构将作为更详细的测试\(如集成测试或验收测试\)的副作用进行介绍。



请求对象的另一种选择是命令。我们可以使用多种应用程序方法设计服务。它们中的每一个都带有您在请求中放入的参数。它适用于简单的应用程序。别担心，我们稍后再来讨论这个话题。

