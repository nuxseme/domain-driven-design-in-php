Validating object compositions can be complicated, because of this, the preferred way of achieving this is through a Domain Service. The service then communicates with repositories in order to retrieve the valid Aggregate. Due to the likely complex object graphs that can be created, an Aggregate could be in an intermediate state, requiring other aggregates to be validated before-hand. We can use Domain Events to notify other parts of the system that a particular element has been validated.



