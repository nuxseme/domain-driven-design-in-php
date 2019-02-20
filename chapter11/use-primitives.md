We recommend using just basic types to build up your request objects. That means string, integers, booleans, and so on. We are just abstracting away input parameters. You should be able to consume Application Services independently from the delivery mechanism. Even pretty complicated HTML forms get translated into basic types all the time at the controller level. You don’t want to mangle your framework and your business logic together.

On some scenarios is tempting to use Value Objects directly. Don’t do it, updates on the Value Object definition will affect all clients. You’ll be coupling clients with your Domain logic.



我们建议只使用基本类型来构建请求对象。这意味着字符串、整数、布尔值等等。我们只是抽象出输入参数。您应该能够独立于交付机制使用应用程序服务。即使是非常复杂的HTML表单，也会在控制器级别上一直转换为基本类型。您不想将框架和业务逻辑混为一谈。

在某些场景中，直接使用值对象是很诱人的。不要这样做，值对象定义的更新将影响所有客户机。您将把客户端与域逻辑耦合起来。

