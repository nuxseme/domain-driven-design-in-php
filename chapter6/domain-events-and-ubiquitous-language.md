Consider the differences in the Ubiquitous Language when we discuss the side effects from relocating a customer, the event makes the concept explicit where as previously the changes that would occur within an aggregate or between multiple aggregates were left as an implicit concept that needed to be explored and defined. As an example, in most systems the fact that a side effect occurred is simply found by a tool such as Hibernate or Entity Framework, if there is a change to the side effects of a use case, it is an implicit concept. The introduction of the event makes the concept explicit and part of the Ubiquitous Language; relocating a customer does not just change some stuff, relocating a customer produces a CustomerRelocatedEvent which is explicitly defined within the language.



