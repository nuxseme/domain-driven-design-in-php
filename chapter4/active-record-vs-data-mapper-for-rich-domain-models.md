Every project always face the decision of which ORM should use. There are a lot of good ORMs for PHP out there: Doctrine, Propel, Eloquent, Paris and many many more.

Most of them are Active Record² implementations. An Active Record implementation is fine mostly for CRUD applications, but it’s not the ideal solution for rich domain models, for the following reasons



The Active Record pattern assumes a one-to-one relation between an entity and a database table. So it couples the design of the database to the design of the object system. And in a rich domain model sometimes entities are constructed with information that may come from different data sources.

Advanced things like collections or inheritance are tricky to implement

Most of the implementations force the use, through inheritance, of some sort of constructions to impose several conventions. This can lead to persistence leakage into the domain model by coupling the domain model with the ORM. The only Active Record implementation the author has seen that does not impose inheriting from a base classes is Castle ActiveRecord³ from Castel Project⁴, a .NET framework. While this leads to some degree of separation between persistence and domain concerns in the produced entities, it doesn’t prevent from coupling the data design with the objects design.



