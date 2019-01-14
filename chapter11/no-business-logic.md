Avoid to put any business logic inside you request objects. Not even validation. Validation should happen inside your Domain – this is inside your Entities, Value Objects, Domain Services, etc. – as business invariants and constraints.

