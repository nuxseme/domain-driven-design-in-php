Hexagonal Architecture is a good foundational architecture but it has some limitations. For example, complex UIs can require aggregate information displayed in diverse forms or they can require data obtained from multiple aggregates. And in this scenario, we can end up with a lot of finder methods inside the Repositories \(maybe as many as the UI views which exist within the application\). Or maybe we can decide to move this complexity to the Application Services - using complex structures to accumulate data from multiple aggregates. Here is an example:



六边形架构是一种非常好的基础架构，但是它也有很多限制。例如，复杂的UI可能需要大量的聚合数据，在这个场景下，我们可能就在很多的查找Repositories的方法中迷失。或者我们将这些复杂性转移到Appliocation Services 处理，使用复杂的结构去组织数据：

```php
interface PostRepository
{

    public function save(Post $post);

    public function byId(PostId $id);

    public function all();

    public function byCategory(CategoryId $categoryId);

    public function byTag(TagId $tagId);

    public function withComments(PostId $id);

    public function groupedByMonth();

    // ...

}
```

When these techniques are abused, the construction of the UI views can become really painful and we should evaluate the trade-offs between making Application Services return domain model instances and using some kind of Data Transfer Object \(DTO\) in order to avoid tight coupling between the Domain Model and infrastructure code like web controllers, CLI controllers, and so on. Luckily, there is another approach. If the problem is having multiple and disparate views, we can exclude them from the Domain Model and start treating them as a purely infrastructural concern. This option is based on a design principle, named Command Query Separation CQS, defined by Bertrand Meyer which gave birth to a new architectural pattern named Command Query Responsibility Segregation defined by Greg Young.



当这些众多的find方法被滥用的时候，UI结构会变得非常复杂和痛苦。我们应该评估，由Application Services 转向Domain Model，同时使用DTO来避免Domain Model和其他层,controller,CLI controller 的直接交流。幸运的是这里有另外一个途径。如果问题是有多个不同的视图，我们可以将它们从域模型中排除，并开始将它们视为纯粹的基础设施关注点。这个选项基于一个设计原则，名为命令查询分离CQS，它产生了一个新的架构模式，名为命令查询责任隔离，由Greg Young定义





> Command Query Separation \(CQS\)
>
> “Asking a question should not change the answer” – Bertrand Meyer This design principle states that every method should be either a Command, that performs an action, or a Query, that returns data to the caller, but not both.



> 提出一个问题不应该改变答案”——Bertrand Meyer这个设计原则指出，每个方法都应该是一个命令，执行一个动作，或者是一个查询，返回数据给调用者，但不能两者兼而有之



CQRS seeks an even more aggressive separation of concerns splitting the Model in two:

• The Write Model: Also known as the Command Model, it performs the writes and takes responsibility for the true domain behaviour.

• The Read Model: It takes responsibility of the reads within the application and treats them as something that should be out of the Domain Model.



CQRS寻求更激进的关注点分离，将模型一分为二:

•写模型:也称为命令模型，它执行写操作并对真正的域行为负责。

•读取模型:它负责应用程序内的读取，并将它们视为域模型之外的内容。



Every time someone triggers a command to the write model, this performs the write to the desired datastore and additionally triggers the read model update in order to display the latest changes on the read model.This strict separation triggers another problem, Eventual Consistency. The consistency of the read model now is subject to the commands performed by the write model. In other words, it is said that the read model is eventually consistent. This is, every time the write model performs a command it will pull up a process that will be responsible to update the read model according to the last updates on the write model. There is such a window of time were the UI may present stale information to the user. In the web scenario this happens often as we are somewhat limited by the current technologies.Think about a caching system in front of a web application. Every time the database is updated with new information, the data on the cache layer may potentially be stale, so every time it gets updated there should be a process that updates the cache system. Cache systems are eventually consistent.This kind of processes, speaking in CQRS terminology, are called Write Model Projections or just Projections. We project the write model onto the read model. This process can be synchronous or asynchronous, depending on your needs, and it can be done thanks to another useful tactical design pattern that will be explained in detail later on in the book, Domain Events. The basis of the write model projections is to gather all the published domain events and update the read model with all the information coming from the events.



每当有人对写模型触发一个命令时，就执行对所需数据存储的写操作，并另外触发读模型更新，以便在读模型上显示最新的更改。这种严格的分离引发了另一个问题，最终的一致性。读模型的一致性现在取决于写模型执行的命令。换句话说，据说读取模型最终是一致的。也就是说，每次写模型执行命令时，它都会拉出一个进程，该进程负责根据写模型上的最新更新更新读模型。如果UI可能向用户提供过时的信息，就会出现这样的时间窗口。在web场景中，这种情况经常发生，因为我们在某种程度上受到当前技术的限制。考虑web应用程序前面的缓存系统。每次数据库更新新信息时，缓存层上的数据可能会过时，因此每次更新时，都应该有一个进程更新缓存系统。缓存系统最终是一致的。这种过程，用CQRS术语来说，被称为写模型预测或者仅仅是预测。我们将写模型投影到读模型上。这个过程可以是同步的，也可以是异步的，这取决于您的需要，并且可以通过另一种有用的策略设计模式来实现，该模式将在本书稍后的“Domain Events”中详细介绍。写模型预测的基础是收集所有已发布的域事件，并使用来自事件的所有信息更新读模型

