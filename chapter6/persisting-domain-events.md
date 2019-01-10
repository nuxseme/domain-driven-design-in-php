

Persisting events is always a good idea. Some readers may be thinking why not publishing Domain Events to a messaging or logging system directly, but persisting them has interesting benefits:



*     You can expose your Domain Events for other BC in a REST way
*     You can persist the Domain Event and the Aggregate changes in the same Database transaction before pushing it to RabbitMQ \(You don?t want to notify about something that did not happen. You don?t want to miss a notification about something that did happen\)
*     Business Intelligence can use this data to analyse, forecast or trend
*     Audit your entity changes
*     For Event Sourcing, you can reconstitute Aggregates from Domain Events



