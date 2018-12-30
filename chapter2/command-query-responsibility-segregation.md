Hexagonal Architecture is a good foundational architecture but it has some limitations. For example, complex UIs can require aggregate information displayed in diverse forms or they can require data obtained from multiple aggregates. And in this scenario, we can end up with a lot of finder methods inside the Repositories \(maybe as many as the UI views which exist within the application\). Or maybe we can decide to move this complexity to the Application Services - using complex structures to accumulate data from multiple aggregates. Here is an example:

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

> Command Query Separation \(CQS\)
>
> “Asking a question should not change the answer” – Bertrand Meyer This design principle states that every method should be either a Command, that performs an action, or a Query, that returns data to the caller, but not both.

CQRS seeks an even more aggressive separation of concerns splitting the Model in two:

• The Write Model: Also known as the Command Model, it performs the writes and takes responsibility for the true domain behaviour.

• The Read Model: It takes responsibility of the reads within the application and treats them as something that should be out of the Domain Model.

Every time someone triggers a command to the write model, this performs the write to the desired datastore and additionally triggers the read model update in order to display the latest changes on the read model.This strict separation triggers another problem, Eventual Consistency. The consistency of the read model now is subject to the commands performed by the write model. In other words, it is said that the read model is eventually consistent. This is, every time the write model performs a command it will pull up a process that will be responsible to update the read model according to the last updates on the write model. There is such a window of time were the UI may present stale information to the user. In the web scenario this happens often as we are somewhat limited by the current technologies.Think about a caching system in front of a web application. Every time the database is updated with new information, the data on the cache layer may potentially be stale, so every time it gets updated there should be a process that updates the cache system. Cache systems are eventually consistent.This kind of processes, speaking in CQRS terminology, are called Write Model Projections or just Projections. We project the write model onto the read model. This process can be synchronous or asynchronous, depending on your needs, and it can be done thanks to another useful tactical design pattern that will be explained in detail later on in the book, Domain Events. The basis of the write model projections is to gather all the published domain events and update the read model with all the information coming from the events.

