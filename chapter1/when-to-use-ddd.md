Domain-Driven Design is not a silver bullet, as everything in software, it depends on the context.  
As a rule of thumb, use it to simplify your domain, never to add more complexity.  
If your application is data-centric and use-cases evolve around data manipulation and CRUD  
operations - this is, Create, Read, Update and Delete - you do not need DDD. The only thing your  
company needs is a fancy face in front of your database.  
If your application has less than 30 use-cases, it might be simpler to use a framework like Symfony  
or Laravel to handle your business logic.  
If you have more than 30 use-cases, your system maybe moving towards the dredded ‘big ball of  
mud’. If you know for sure your system will grow in complexity, you should start considering using  
DDD to fight complexity

If you know your application is gonna grow and is likely to change often, DDD will definitively help  
in managing the complexity and refactoring your model over time.  
If you do not understand the Domain you are working on because it is new and nobody invested  
on a solution before, this might mean it is complex enough to start applying DDD. You will need to  
work closely with Domain Experts to get the models right.

领域驱动的设计并不是银弹，因为软件中的一切都取决于上下文。根据经验，使用它来简化您的领域，永远不要增加更多的复杂性。如果您的应用程序是以数据为中心的，并且用例围绕数据操作和CRUD操作,创建、读取、更新和删除,你不需要DDD。你的公司唯一需要的就是面向数据库。如果应用程序的用例少于30个，那么使用Symfony或Laravel之类的框架来处理业务逻辑可能会更简单。如果您有超过30个用例，那么您的系统可能会转向“大泥球”。如果您确信您的系统将变得越来越复杂，那么您应该开始考虑使用DDD来对抗复杂性，如果您知道您的应用程序将会增长并且可能会经常改变，那么DDD将在管理复杂性和重构您的模型方面起到决定性的作用。如果您不了解您正在处理的领域，因为它是新的，而且以前没有人在解决方案上实践过，那么这可能意味着它足够复杂，可以开始应用DDD。您将需要与领域专家密切合作以获得正确的模型。

---

1. 没有架构设计是银弹，特定的环境采用不同的方案
2. 简单系统直接面向数据库编程
3. 轻系统可以应用集成的框架
4. 简单高效的解决问题，不应该炫技将系统变得更复杂，请克制你的虚荣心



