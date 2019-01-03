Whilst modelling an Ubiquitous Languages concept in code, you should always favour Value Objects over Entities. Value Objects are easier to create, test, use and maintain.

With this in mind, you can decide on whether the concept in question could be modelled as a Value Object if…

It measures, quantifies, or describes a thing in the domain

It can be kept immutable

It models a conceptual whole, by composing related attributes as an integral unit

It is completely replaceable when the measurement or description changes

It can be compared with others through value equality

It supplies its collaborators with Side-Effect-Free behaviour

