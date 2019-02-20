Domain Events should be published when the fact they represent happens. For instance, when a new user has been registered, a new UserRegistered event should be published.

Following the newspaper metaphor:

* Modeling a Domain Event is like writing a news article
* Publishing a Domain Event is like printing the article in the paper
* Spreading a Domain Event is like sending the newspaper so everyone can read the article

The recommended approach for publishing DomainEvents is to use a simple Listener-Observer pattern to implement a DomainEventPublisher.



域事件应该在它们所表示的事实发生时发布。例如，当注册了一个新用户时，应该发布一个新的UserRegistered事件。

以下是报纸的比喻:

* 建模域事件就像编写一篇新闻文章
* 发布域事件就像在论文中打印文章一样
* 传播域事件就像发送报纸，这样每个人都可以阅读文章

发布DomainEvents的推荐方法是使用简单的侦听器-观察者模式来实现DomainEventPublisher。

