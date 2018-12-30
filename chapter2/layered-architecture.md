From the code maintainability and reuse perspectives, the best way to make this code a bit easier to maintain would be splitting up concepts - creating layers for each different concern. In our previous example, it is easy to shape some different layers like the one encapsulating the data access and manipulation, another one to handle infrastructure concerns and a final one for encapsulating the orchestration of the previous two. An essential rule of the Layered architecture is that each layer may be tightly coupled with the layers beneath it, as shown in the following picture

> 从代码的可维护性和重用性的角度来看，使代码更容易维护的最好办法是分离概念，为不同的关注点分层。在上一个例子中，很容易形成不同的层，访问数据的封装层，基础设施访问层。分层体系的一个重要的规则是每个层只关注该层概念，只与其他下层紧密耦合

![](/assets/layered-architecture.png)

What the layered architecture really seeks is the separation of the different components of an application. For instance, in terms of the previous example, a blog post representation must be completely independent of a blog post as a conceptual entity. A blog post as a conceptual entity can instead be associated with one or more representations, as opposed to being tightly coupled to a specific representation. This is commonly named Separation of Concerns.Another architecture paradigm and pattern that seeks the same purpose is the Model-View-Controller pattern. It was initially thought and widely-used for building desktop GUI applications, and now it is mainly used in web applications thanks to popular web frameworks like Symfony, Zend Framework or Codeigniter.

> 分层架构真正寻求的是应用程序不同组件的分离。例如，博客文章表示必须完全独立于作为概念实体的博客文章。博客文章作为一个概念实体可以与一个或多个表示相关联，而不是与特定的表示紧密耦合。这通常称为关注点隔离。另一个寻求相同目的的体系结构范式和模式是模型-视图-控制器模式。它最初是用于构建桌面GUI应用程序的，现在主要用于web应用程序，这要归功于一些流行的web框架，如Symfony、Zend Framework或Codeigniter。

---

* 解决方案最终还是解决人脑智力不足的问题



