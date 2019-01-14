If we take the example of a fictional e-commerce application, named buy.it it may make sense to define a module for each of the different bounded contexts that compose the e-commerce application, so each bounded context represents a self-contained and independent application



```
├── billing
│       ├── composer.json
│       ├── composer.lock
│ ├── src
│ └── tests
├── cart
│       ├── composer.json
│       ├── composer.lock
│ ├── src
│ └── tests
├── catalog
│       ├── composer.json
│       ├── composer.lock
│ ├── src
│      └── tests
└──  inventory
├──  composer.json
├──   composer.lock
├── src
└── tests
```

Each module contains an application that exposes a REST-like API. Beware that each module name represents a meaningful concept in an e-commerce system and is named in terms of the Ubiquitous Language:



Billing module to hold all the code related to the payments, bills, waybills, order-processing systems with finite-state machine to be able to process the orders and so on.

* Cart module to hold all the code related to the cart system.
* Catalog module to hold all the code related to the product descriptions, product combinations and so on.
* Inventory module to hold all the code related to the management of product stocks.



Let’s dig a bit further into one of those modules. Let’s take for example the Billing context and examine the structure details. As its name suggests this module is responsible for representing all the flows that an order passes. From its creation until it’s delivered to the customer who has purchased it. Furthermore, it’s an independent application, so it contains a source code folder and a tests folder. The source code folder contains all the code necessary for this bounded context to work: domain code and infrastructure code.

```
├──  composer.json
├──   composer.lock
├── src
│ └── BuyIt
│	└── Billing
│	├── DomainModel
│	└── Infrastructure
└── tests
```



All the code is prefixed with a vendor namespace named in terms of the organization name \(BuyIt, in this case\) and contains two subfolders: DomainModel holds all the domain code and Infrastructure holds the infrastructure layer, isolating all the domain logic from the details of the infrastructure layer. Following this structure we’re making clear that we’re going to use Hexagonal Architecture as a foundational architecture. An alternative structure we may have used, would be one as the following

```
├──  composer.json
├──   composer.lock
├── src
│ └── BuyIt
│	└── Billing
│	├── Domain
│	│ ├── Model
│	│ └── Service
│	└── Infrastructure
└── tests

```



This style of structure uses an additional subfolder to store the services defined inside the domain model. While this organization may make sense, our preference here is to tend not to use it, since this way of separating code tends to be more focused on the architectural elements rather than the relevant concepts in the model. We believe that this style could easily lead to some sort of service layer on top of the domain model and this is not necessary a bad thing, but keep in mind that Domain Services are used to describe things into the domain, operations that don’t belong to entities nor value objects. So, from now on we will stick with the previous code organization.



> It’s possible to place code inside the DomainModel subfolder directly. For example, it may be quite common to place common interfaces and services in it, like the DomainEventPub- lisher, the DomainEventSubscriber and so on.



If we had to model a billing context, probably we would have an Order entity with its repository and all the state information. So our first attempt would be to directly place all those elements directly into the DomainModel subfolder. At a first glance, this may seem the simplest way

```
├──  composer.json
├──   composer.lock
├── src
│ └── BuyIt
│	└── Billing
│	├── DomainModel
│	│ ├── Order.php
│	│ ├── OrderLine.php
│	│ ├── OrderLineWasAdded.php
│	│        ├──  OrderRepository.php
│	│      └── OrderWasCreated.php
│	└── Infrastructure
└── tests

```





We’ve placed the Order and the OrderLine entities, the OrderLineWasAdded and the OrderWasCre- ated event and the OrderRepository into the same subfolder \(DomainModel\). This structure may be fine, but that’s because we still have a simple model. What about the Bill entity plus its repository? Or the Waybill entity plus its respective repository? Let’s add all those elements, and see how it fits into the actual code structure



```
├──  composer.json
├──   composer.lock
├── src
│ └── BuyIt
│	└── Billing
│	├── DomainModel
│	│ ├── Bill.php
│	│ ├── BillLine.php
│	│ ├── BillLineWasAdded.php
│	│         ├──  BillRepository.php
│	│        ├──  BillWasCreated.php
│	│ ├── Order.php
│	│ ├── OrderLine.php
│	│ ├── OrderLineWasAdded.php
│	│        ├──  OrderRepository.php
│	│      ├── OrderWasCreated.php
│	│ ├── Waybill.php
│	│ ├── WaybillLine.php
│	│ ├── WaybillLineWasAdded.php
│	│ ├── WaybillRepository.php
│	│ └── WaybillWasGenerated.php
│	└── Infrastructure
└── tests
```



While this style of code organization could be fine, it can become non-practical and pretty unmaintainable in the long term. Every time we iterate and add new features, the model will become even more bigger and that subfolder will be eating even more code. We’re in the need to split the code in a way that give us a perspective of the model at a glance. No technical concerns, just domain concerns. To reach this, we can split this model using the Ubiquitous Language, finding meaningful concepts that help us group elements logically in terms of the domain. So we could try an approach as the following

```
├──  composer.json
├──   composer.lock
├── src
│ └── BuyIt
│	└── Billing
│	├── DomainModel
│	│ ├── Bill
│	│ │ ├── Bill.php
│	│ │ ├── BillLine.php
│	│ │ ├── BillLineWasAdded.php
│	│        │        ├── BillRepository.php
│	│       │       └── BillWasCreated.php
│	│ ├── Order
│	│ │ ├── Order.php
│	│ │ ├── OrderLine.php
│	│ │ ├── OrderLineWasAdded.php
│	│       │       ├──  OrderRepository.php
│	│      │      └── OrderWasCreated.php
│	│ └── Waybill
│	│	├── Waybill.php
│	│	├── WaybillLine.php
│	│	├── WaybillLineWasAdded.php
│	│	├── WaybillRepository.php
│	│	└── WaybillWasGenerated.php
│	└── Infrastructure
└── tests
```



This way the code is more organized, conceptually speaking. And not only that. As Evans points out the blue book¹, Modules are a way to communicate as they give us insights about how the domain model works internally, and help us increase the cohesion and decrease the coupling between the concepts. If we look at the previous example, we can see that the concepts Order and OrderLine are strongly related so they live in the same module. On the other hand, Order and Waybill although sharing the same context, they are different concepts so they live in different modules. Modules are not just a way to group related concepts into the model but a way to express part of the design of the model.



> Should we place Repositories, Factories, Domain Events, Services in their own subfolder?
>
> Effectively they could be placed into their own subfolder, but it’s strongly discouraged. Just because this way we would be mixing technical concerns and domain concerns, and remember that the Module’s main interest is to group related concepts from the domain model and decouple them from the non-related. Modules don’t separate code but separate meaningful concepts.





