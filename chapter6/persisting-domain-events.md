Persisting events is always a good idea. Some readers may be thinking why not publishing Domain Events to a messaging or logging system directly, but persisting them has interesting benefits:

持久化事件总是一个好主意。一些读者可能会想，为什么不直接将域事件发布到消息传递或日志系统中，但是持久化它们有一些有趣的好处:

* You can expose your Domain Events for other BC in a REST way
* You can persist the Domain Event and the Aggregate changes in the same Database transaction before pushing it to RabbitMQ \(You don?t want to notify about something that did not happen. You don?t want to miss a notification about something that did happen\)
* Business Intelligence can use this data to analyse, forecast or trend
* Audit your entity changes
* For Event Sourcing, you can reconstitute Aggregates from Domain Events

* 您可以以REST方式为其他BC公开域事件

* 在将域事件和聚合更改推送到RabbitMQ之前，您可以在相同的数据库事务中持久保存域事件和聚合更改\(不想把没有发生的事告诉任何人。不想错过已经发生的事情的通知）

* 商业智能可以使用这些数据来分析、预测或趋势
* 审计实体更改
* 对于事件来源，您可以从域事件重新构建聚合



