How can we fix this? As the Domain Model layer depends on concrete infrastructure implementations,

the Dependency Inversion Principle⁷ could be applied by relocating the Infrastructure layer on

top of the other three layers.



> The Dependency Inversion Principle
>
> High level modules should not depend upon low level modules. Both should depend upon
>
> abstractions.
>
> Abstractions should not depend upon details. Details should depend upon abstractions.
>
> Robert C. Martin



By using the Dependency Inversion Principle, the architecture schema changes and the Infrastructure

layer which can be referred to as low level modules now depend on the UI, the Application Layer

and the Domain Layer, which are the high level modules. The dependency has been inverted.

But then, what is Hexagonal Architecture?, and how does it fit within of all this? Hexagonal

Architecture \(also known as Ports and Adapters\) was defined by Alistair Cockburn and represents

the application as an hexagon where each side represents a Port with one or more Adapters. A

Port is a connector with a pluggable Adapter which transforms an outside input to something the

inside application can understand. In terms of the DIP, the Port would be a high level module and an

Adapter would be a low level module. Furthermore, if the application needs to emit some message to

the outside it will also use a Port with an Adapter to send it and transform it to something that the

outside can understand. For this reason, Hexagonal Architecture brings up the concept of symmetry

in the application and it is also the main reason why the schema of the architecture changes. It is

often represented as a hexagon, because it does no longer make sense to talk about a “top” layer

nor a “bottom” layer. Instead, Hexagonal Architecture talks mainly in terms of the ‘outside’ and the

‘inside’.

