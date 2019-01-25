In a Ticket Sales Agency, a content manager decides to increase the price of a U2 show. Using her back-office, edits the show. A ShowPriceChanged Domain Event is published and persisted in the same transaction with the new show price into the database.

A batch process takes the Domain Event and queues it into RabbitMQ. The Domain Event gets distributed in two queues, one for the same local Bounded Context and another remote one for Business Intelligence purposes.

In the first queue, a worker fetches the corresponding Show using the id in the event and push it into an Elasticsearch server, so the user can see the new price when searching. It could also update the new price in a different database table.

In the second queue, a worker inserts the info into a Logs Server or a Data Lake where reporting or Data Mining processes can be run.

An external application that cannot be integrated using Domain Events, could access to all the

ShowPriceChanged events using a REST API that the local Bounded Context provides.

As you can see, Domain Events are useful for dealing with eventual consistency and integrating different Bounded Contexts. Aggregates create Events and publish them. Subscribers may store Events and then forward them to remote subscribers.



在门票销售代理机构，内容经理决定提高U2演出的价格。利用她的后台办公室，编辑节目。将发布一个ShowPriceChanged域事件，并将该事件与新的show price保存在数据库中的相同事务中。

批处理流程接受域事件并将其排队到RabbitMQ中。域事件分布在两个队列中，一个用于相同的本地有界上下文，另一个用于业务智能目的的远程队列。

在第一个队列中，工作人员使用事件中的id获取相应的Show，并将其推入Elasticsearch服务器，这样用户在搜索时就可以看到新的价格。它还可以在不同的数据库表中更新新的价格。

在第二个队列中，工作人员将信息插入日志服务器或数据池中，可以在其中运行报告或数据挖掘流程。

不能使用域事件集成的外部应用程序可以访问所有

ShowPriceChanged事件使用本地有界上下文提供的REST API。

如您所见，域事件对于处理最终一致性和集成不同的有界上下文非常有用。聚合创建事件并发布它们。订阅者可以存储事件，然后将它们转发给远程订阅者。

---

1. 生产者只关心事件发布
2. 消费者只关心自己感兴趣的事件



