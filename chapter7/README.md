When you place some classes together in a Module, you are telling the next developer who looks at your design to think about them together. If your model is telling a story, the Modules are chapters.

当您将一些类放在一个模块中时，您是在告诉下一个查看您的设计的开发人员一起考虑它们。如果您的模型正在讲述一个故事，那么模块就是章节。

Eric Evans, Domain-Driven Design

A common concern when building an application following DDD, is where do we put the code? What’s the recommended way to place the code into the application? Where do we place infrastructure code? And more important, how should the different concepts inside the model be structured?

在使用DDD构建应用程序时，一个常见的问题是，我们将代码放在哪里?将代码放入应用程序的推荐方法是什么?我们把基础架构代码放在哪里?更重要的是，模型中不同的概念应该如何构建?

There’s a tactical pattern for this: modules. Nowadays, everyone structures the code in modules. But DDD goes ones step further and no technical concerns are considered when using modules. Indeed, it treats modules as a part of the model.

这里有一个战术模式:模块。现在，每个人都在模块中构造代码。但是DDD更进一步，在使用模块时不考虑技术问题。实际上，它将模块视为模型的一部分。

> Modules should not be treated as a way to separate code but as a way to separate meaningful concepts in the model.
>
> 模块不应该被视为分离代码的一种方法，而应该被视为分离模型中有意义的概念的一种方法。



