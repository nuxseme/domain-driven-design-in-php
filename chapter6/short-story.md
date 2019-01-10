In a Ticket Sales Agency, a content manager decides to increase the price of a U2 show. Using her back-office, edits the show. A ShowPriceChanged Domain Event is published and persisted in the same transaction with the new show price into the database.

A batch process takes the Domain Event and queues it into RabbitMQ. The Domain Event gets distributed in two queues, one for the same local Bounded Context and another remote one for Business Intelligence purposes.

In the first queue, a worker fetches the corresponding Show using the id in the event and push it into an Elasticsearch server, so the user can see the new price when searching. It could also update the new price in a different database table.

In the second queue, a worker inserts the info into a Logs Server or a Data Lake where reporting or Data Mining processes can be run.

An external application that cannot be integrated using Domain Events, could access to all the

ShowPriceChanged events using a REST API that the local Bounded Context provides.

As you can see, Domain Events are useful for dealing with eventual consistency and integrating different Bounded Contexts. Aggregates create Events and publish them. Subscribers may store Events and then forward them to remote subscribers.



