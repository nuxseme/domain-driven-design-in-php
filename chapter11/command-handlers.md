An interesting way of executing Application Services is through a Command Bus library. A good one is Tactician¹⁰

> What is a Command Bus? The term is mostly used when we combine the Command pattern¹¹ with a service layer¹². Its job is to take a Command object \(which describes what the user wants to do\) and match it to a Handler \(which executes it\). This can help structure your code neatly.

Fair enough, our Application Services are the Service Layer and our Request objects looks pretty much like Commands.

Wouldn’t be great if we had a mechanism to link all the Application Services and then based on the Request execute the correct one? Well, that is actually what a Comment Bus is.



执行应用程序服务的一种有趣的方式是通过命令总线库。一个好的是战术家



> 什么是命令总线?当我们将命令模式与服务层结合使用时，通常会使用这个术语。它的工作是获取一个Command对象\(描述用户想要做什么\)并将其与一个Handler\(执行它\)匹配。这有助于整齐地组织代码。

很好，我们的应用程序服务是服务层，我们的请求对象看起来很像命令。



如果我们有一种机制来链接所有的应用程序服务，然后根据请求执行正确的服务，这不是很好吗?这就是注释总线。

