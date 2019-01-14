The mapping between your domain objects and the database can be considered an implementation detail. The domain life-cycle should not be aware of these persistence details. As such, the mapping information should be defined as part of the infrastructure layer, outside the domain and as the implementation for the repositories.

