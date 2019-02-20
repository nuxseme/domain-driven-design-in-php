The Application layer is the area that separates the Domain Model from the clients that query or change its state. Application Services are the building blocks for such layer. As Vaughn Vernon says, “Application Services are the direct clients of the domain model”. You could think about an Application Service as a point of contact between the outside world \(html forms, API clients, command line, frameworks, UI, etc.\) and the Domain Model itself. It might help thinking about the top level use cases that your system exposes to the world “as guest, I want to register”, “as a logged user, I want to purchase a product”, etc.

In this chapter, we will explore how to implement Application Services, understanding the role of the Command Pattern and establishing the responsibilities of an Application Service. Consider the use case of signing up a new user.

应用层是将域模型与查询或更改其状态的客户机分隔开的区域。应用程序服务是这一层的构建块。正如Vaughn Vernon所说，“应用程序服务是域模型的直接客户端”。您可以将应用程序服务看作外部世界\(html表单、API客户机、命令行、框架、UI等\)和域模型本身之间的接触点。这可能有助于思考您的系统向世界公开的顶级用例“作为客户，我想注册”、“作为已登录用户，我想购买产品”等等。

在本章中，我们将探讨如何实现应用程序服务、理解命令模式的作用以及确定应用程序服务的职责。考虑注册新用户的用例。

Conceptually, in order to register a new user we need to:

* Get an email and password from the client
* Check if the email is already in use
* Create a new user
* Add this new user to the existing user set
* Return the user we’ve just created 

Let’s go for it.

从概念上讲，为了注册新用户，我们需要:

* 从客户端获取电子邮件和密码
* 检查邮件是否已经在使用中
* 创建一个新用户
* 将此新用户添加到现有用户集
* 返回我们刚刚创建的用户

让我们开始吧。

