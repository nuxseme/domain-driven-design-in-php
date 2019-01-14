So far we have been discussing how we structure and organize code in the domain layer, but we’ve almost said nothing about the Infrastructure Layer. And, since we’re using Hexagonal Architecture to inverse the dependency between the domain and the infrastructure layer, we will need a place where we can put all the implementations of the interfaces defined in the domain layer. Returning to the example of the billing context, we need a place for the implementations of BillRepository, OrderRepository and WaybillRepository.

It’s clear that they should be placed into the Infrastructure folder, but where? Suppose we decided to use Doctrine ORM to implement the persistence layer. How do we put the Doctrine implementations of our repositories into the Infrastructure folder? Let’s put it directly into it and see how it looks.

```
├── composer.json
├── composer.lock
├── src
│ └── BuyIt
│ └── Billing
│ ├── DomainModel
│ │ ├── Bill
│ │ │ ├── Bill.php
│ │ │ ├── BillLine.php
│ │ │ ├── BillLineWasAdded.php
│ │ │ ├── BillRepository.php
│ │ │ └── BillWasCreated.php
│ │ ├── Order
│ │ │ ├── Order.php
│ │ │ ├── OrderLine.php
│ │ │ ├── OrderLineWasAdded.php
│ │ │ ├── OrderRepository.php
│ │ │ └── OrderWasCreated.php
│ │ └── Waybill
│ │ ├── Waybill.php
│ │ ├── WaybillLine.php
│ │ ├── WaybillLineWasAdded.php
│ │ ├── WaybillRepository.php
│ │ └── WaybillWasGenerated.php
│ └── Infrastructure
│ ├── DoctrineBillRepository.php
│ ├── DoctrineOrderRepository.php
│ └── DoctrineWaybillRepository.php
└── tests
```

We could leave it as this. But as we have seen in the Domain Layer, this structure and organization

will rot fast and become a mess within a few model iterations. Each time the model grows, it will

probably need even more infrastructure so this way we will end up mixing different technical

concerns such as persistence, messaging, logging and a large etcetera. Our first attempt to avoid

a tangled mess of infrastructure implementations is to define a module for each technical concern

into the bounded context.

```
├── composer.json
├── composer.lock
├── src
│ └── BuyIt
│ └── Billing
│ ├── DomainModel
│ │ ├── Bill
│ │ │ ├── Bill.php
│ │ │ ├── BillLine.php
│ │ │ ├── BillLineWasAdded.php
│ │ │ ├── BillRepository.php
│ │ │ └── BillWasCreated.php
│ │ ├── Order
│ │ │ ├── Order.php
│ │ │ ├── OrderLine.php
│ │ │ ├── OrderLineWasAdded.php
│ │ │ ├── OrderRepository.php
│ │ │ └── OrderWasCreated.php
│ │ └── Waybill
│ │ ├── Waybill.php
│ │ ├── WaybillLine.php
│ │ ├── WaybillLineWasAdded.php
│ │ ├── WaybillRepository.php
│ │ └── WaybillWasGenerated.php
│ └── Infrastructure
│ ├── Logging
│ ├── Messaging
│ └── Persistence
│ ├── DoctrineBillRepository.php
│ ├── DoctrineOrderRepository.php
│ └── DoctrineWaybillRepository.php
└── tests
```

This looks much better and is a lot more maintainable in the long term than our first attempt. And

if you know beforehand that you will always have a single persistence mechanism, you can stick

with this structure and organization. It’s quite simple and easy to maintain. But what about when

you have to play with several persistence mechanisms? Nowadays, it’s quite common to have a

relational one, and some kind of shared in-memory persistence like Redis or Riak. Or to have some

sort of local in-memory implementation to be able to test the code. Let’s see how this fits into the actual approach.

```
├── composer.json
├── composer.lock
├── src
│ └── BuyIt
│ └── Billing
│ ├── DomainModel
│ │ ├── Bill
│ │ │ ├── Bill.php
│ │ │ ├── BillLine.php
│ │ │ ├── BillLineWasAdded.php
│ │ │ ├── BillRepository.php
│ │ │ └── BillWasCreated.php
│ │ ├── Order
│ │ │ ├── Order.php
│ │ │ ├── OrderLine.php
│ │ │ ├── OrderLineWasAdded.php
│ │ │ ├── OrderRepository.php
│ │ │ └── OrderWasCreated.php
│ │ └── Waybill
│ │ ├── Waybill.php
│ │ ├── WaybillLine.php
│ │ ├── WaybillLineWasAdded.php
│ │ ├── WaybillRepository.php
│ │ └── WaybillWasGenerated.php
│ └── Infrastructure
│ ├── Logging
│ ├── Messaging
│ └── Persistence
│ ├── DoctrineBillRepository.php
│ ├── DoctrineOrderRepository.php
│ ├── DoctrineWaybillRepository.php
│ ├── InMemoryBillRepository.php
│ ├── InMemoryOrderRepository.php
│ ├── InMemoryWaybillRepository.php
│ ├── RedisBillRepository.php
│ ├── RedisOrderRepository.php
│ └── RedisWaybillRepository.php
└── tests
```

Again this now seems a bit odd. All the repository implementations are living in the same module,

and this leads to a mix of several technologies. For now, we only have a few repositories. With a few

more, the maintainability could start degrading considerably. So this makes the point clear that we

need to create another module, to group the related implementations by the underlying technology.

```
├── composer.json
├── composer.lock
├── src
│ └── BuyIt
│ └── Billing
│ ├── DomainModel
│ │ ├── Bill
│ │ │ ├── Bill.php
│ │ │ ├── BillLine.php
│ │ │ ├── BillLineWasAdded.php
│ │ │ ├── BillRepository.php
│ │ │ └── BillWasCreated.php
│ │ ├── Order
│ │ │ ├── Order.php
│ │ │ ├── OrderLine.php
│ │ │ ├── OrderLineWasAdded.php
│ │ │ ├── OrderRepository.php
│ │ │ └── OrderWasCreated.php
│ │ └── Waybill
│ │ ├── Waybill.php
│ │ ├── WaybillLine.php
│ │ ├── WaybillLineWasAdded.php
│ │ ├── WaybillRepository.php
│ │ └── WaybillWasGenerated.php
│ └── Infrastructure
│ ├── Logging
│ ├── Messaging
│ └── Persistence
│ ├── Doctrine
│ │ ├── DoctrineBillRepository.php
│ │ ├── DoctrineOrderRepository.php
│ │ └── DoctrineWaybillRepository.php
│ ├── InMemory
│ │ ├── InMemoryBillRepository.php
│ │ ├── InMemoryOrderRepository.php
│ │ └── InMemoryWaybillRepository.php
│ └── Redis
│ ├── RedisBillRepository.php
│ ├── RedisOrderRepository.php
│ └── RedisWaybillRepository.php
└── tests

```

This structure and organization of the infrastructure layer is much more maintainable and easier

to understand than our previous attempt. And we can have a general idea about the technologies

being used in this bounded context.

implementation. Take into account that this is just an example and a way to do it. Probably it can

be improved, for example now the whole add operation is synchronous. We could instead enqueue

the operation to some sort of messaging middleware that stores the Order into Elastic, for example.

There are a lot of posibilities and improvements, depending on your needs.

