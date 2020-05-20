So far we have been discussing how we structure and organize code in the domain layer, but we’ve almost said nothing about the Infrastructure Layer. And, since we’re using Hexagonal Architecture to inverse the dependency between the domain and the infrastructure layer, we will need a place where we can put all the implementations of the interfaces defined in the domain layer. Returning to the example of the billing context, we need a place for the implementations of BillRepository, OrderRepository and WaybillRepository.

It’s clear that they should be placed into the Infrastructure folder, but where? Suppose we decided to use Doctrine ORM to implement the persistence layer. How do we put the Doctrine implementations of our repositories into the Infrastructure folder? Let’s put it directly into it and see how it looks.

到目前为止，我们一直在讨论如何在域层中构造和组织代码，但是几乎没有提到基础结构层。而且，由于我们使用六角形体系结构来逆转域和基础结构层之间的依赖关系，我们将需要一个位置来放置在域层中定义的所有接口实现。回到账单上下文的示例，我们需要一个地方来实现BillRepository、OrderRepository和WaybillRepository。

很明显，它们应该放在Infrastructure文件夹中，但是在哪里呢?假设我们决定使用Doctrine ORM来实现持久性层。我们如何将存储库的Doctrine实现放到基础结构文件夹中?我们直接把它放进去看看它是什么样子的。

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
│   ├── DoctrineBillRepository.php
│   ├── DoctrineOrderRepository.php
│   └── DoctrineWaybillRepository.php
└── tests
```

We could leave it as this. But as we have seen in the Domain Layer, this structure and organization

will rot fast and become a mess within a few model iterations. Each time the model grows, it will

probably need even more infrastructure so this way we will end up mixing different technical

concerns such as persistence, messaging, logging and a large etcetera. Our first attempt to avoid

a tangled mess of infrastructure implementations is to define a module for each technical concern

into the bounded context.

我们可以这样。但是正如我们在领域层看到的，这种结构和组织在一些模型迭代中会快速腐烂并变得一团糟。每当模型增长时，它就会增长可能需要更多的基础设施，这样我们就可以混合不同的技术诸如持久性、消息传递、日志记录等问题。我们第一次尝试回避基础设施实现的混乱局面是为每个技术问题定义一个模块在有限的上下文中。

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
│   ├── Logging
│   ├── Messaging
│   └── Persistence
│     ├── DoctrineBillRepository.php
│     ├── DoctrineOrderRepository.php
│     └── DoctrineWaybillRepository.php
└── tests
```

This looks much better and is a lot more maintainable in the long term than our first attempt. And

if you know beforehand that you will always have a single persistence mechanism, you can stick

with this structure and organization. It’s quite simple and easy to maintain. But what about when

you have to play with several persistence mechanisms? Nowadays, it’s quite common to have a

relational one, and some kind of shared in-memory persistence like Redis or Riak. Or to have some

sort of local in-memory implementation to be able to test the code. Let’s see how this fits into the actual approach.

从长远来看，这看起来比我们的第一次尝试要好得多，也更易于维护。和如果您事先知道始终只有一个持久性机制，那么您可以坚持下去有了这样的结构和组织。它非常简单，易于维护。但是什么时候呢您必须使用几种持久性机制?现在，拥有a是很常见的关系存储，以及某种共享内存持久性，比如Redis或Riak。或者吃点一种本地内存实现，能够测试代码。让我们看看这如何与实际方法相适应。

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

这看起来有点奇怪。所有的存储库实现都位于同一个模块中，这就导致了几种技术的混合。目前，我们只有几个存储库。有几个而且，可维护性可能开始显著降低。这说明我们需要创建另一个模块，根据底层技术对相关实现进行分组。

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
│   ├── Logging
│   ├── Messaging
│   └── Persistence
│     ├── Doctrine
│     │ ├── DoctrineBillRepository.php
│     │ ├── DoctrineOrderRepository.php
│     │ └── DoctrineWaybillRepository.php
│     ├── InMemory
│     │ ├── InMemoryBillRepository.php
│     │ ├── InMemoryOrderRepository.php
│     │ └── InMemoryWaybillRepository.php
│     └── Redis
│       ├── RedisBillRepository.php
│       ├── RedisOrderRepository.php
│       └── RedisWaybillRepository.php
└── tests
```

This structure and organization of the infrastructure layer is much more maintainable and easier

to understand than our previous attempt. And we can have a general idea about the technologies

being used in this bounded context.

基础结构层的这种结构和组织更易于维护比我们以前的尝试更能理解。我们可以对这些技术有一个大概的了解在这个有限的上下文中使用。

