You should struggle to publish Domain Events from deeper in the chain. The nearer inside the Entity or the Value Object, the better. As we have seen in the previous section, sometimes this is not easy but the final result is simpler for the clients. We have seen developers publishing Domain Events from the Application Services or Domain Services. That seems an easier approach to implement but drives to an Anemic-Domain Model in the same way when pushing business logic to Domain Services rather than placing it into your Entities.



您应该努力从链的更深处发布域事件。实体或值对象内部越近越好。正如我们在前一节中所看到的，有时这并不容易，但是对于客户机来说，最终的结果更简单。我们看到开发人员从应用程序服务或域服务发布域事件。这似乎是一种更容易实现的方法，但是在将业务逻辑推入域服务而不是将其放入实体时，也会以同样的方式驱动到贫血模型。

