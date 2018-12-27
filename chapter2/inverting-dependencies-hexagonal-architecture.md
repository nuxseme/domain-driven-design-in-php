Following the essential rule of Layered Architecture, there is a risk to end up implementing domain

interfaces that need to make use of infrastructural concerns within the domain model layer.

As an example, the PDORepository from the previous example should be placed in the Domain

Model if we were following MVC. However, placing infrastructural details right in the middle of

our domain violates separation of concerns. This can result in issues, it is hard to avoid violating the

essential rules of Layered Architecture, leading to a style of code which can become hard to test if

the Domain Layer is aware of technical implementations.

