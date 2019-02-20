If we take the example of a fictional e-commerce application, named buy.it it may make sense to define a module for each of the different bounded contexts that compose the e-commerce application, so each bounded context represents a self-contained and independent application

如果我们以一个虚构的电子商务应用程序为例，名为buy。为组成电子商务应用程序的每个不同的有界上下文定义一个模块可能是有意义的，因此每个有界上下文表示一个自包含的、独立的应用程序

```
├── billing
│ ├── composer.json
│ ├── composer.lock
│ ├── src
│ └── tests
├── cart
│ ├── composer.json
│ ├── composer.lock
│ ├── src
│ └── tests
├── catalog
│ ├── composer.json
│ ├── composer.lock
│ ├── src
│ └── tests
└── inventory
├── composer.json
├── composer.lock
├── src
└── tests
```

Each module contains an application that exposes a REST-like API. Beware that each module name represents a meaningful concept in an e-commerce system and is named in terms of the Ubiquitous Language:

每个模块都包含一个公开类rest API的应用程序。注意，在电子商务系统中，每个模块的名称都代表一个有意义的概念，并且使用通用语言进行命名:

Billing module to hold all the code related to the payments, bills, waybills, order-processing systems with finite-state machine to be able to process the orders and so on.

计费模块包含与支付、账单、运单、订单处理系统相关的所有代码，具有有限状态机，能够处理订单等。

* Cart module to hold all the code related to the cart system.
* Catalog module to hold all the code related to the product descriptions, product combinations and so on.
* Inventory module to hold all the code related to the management of product stocks.

* Cart模块保存所有与Cart系统相关的代码。
* Catalog模块保存与产品描述、产品组合等相关的所有代码。
* 库存模块保存所有与产品库存管理相关的代码。

Let’s dig a bit further into one of those modules. Let’s take for example the Billing context and examine the structure details. As its name suggests this module is responsible for representing all the flows that an order passes. From its creation until it’s delivered to the customer who has purchased it. Furthermore, it’s an independent application, so it contains a source code folder and a tests folder. The source code folder contains all the code necessary for this bounded context to work: domain code and infrastructure code.

让我们进一步研究其中一个模块。让我们以账单上下文为例，并检查其结构细节。顾名思义，这个模块负责表示订单传递的所有流。从创建到交付给购买它的客户。此外，它是一个独立的应用程序，因此它包含一个源代码文件夹和一个测试文件夹。源代码文件夹包含这个有界上下文工作所需的所有代码:域代码和基础结构代码。

```
├──  composer.json
├──   composer.lock
├── src
│ └── BuyIt
│    └── Billing
│    ├── DomainModel
│    └── Infrastructure
└── tests
```

All the code is prefixed with a vendor namespace named in terms of the organization name \(BuyIt, in this case\) and contains two subfolders: DomainModel holds all the domain code and Infrastructure holds the infrastructure layer, isolating all the domain logic from the details of the infrastructure layer. Following this structure we’re making clear that we’re going to use Hexagonal Architecture as a foundational architecture. An alternative structure we may have used, would be one as the following

所有代码的前缀都有一个供应商名称空间，该名称空间根据组织名称命名\(在本例中是BuyIt\)，并包含两个子文件夹:DomainModel包含所有域代码，基础结构包含基础结构层，将所有域逻辑与基础结构层的细节隔离开来。按照这个结构，我们明确表示我们将使用六边形结构作为基础结构。我们可能使用的另一种结构如下所示

```
├──  composer.json
├──   composer.lock
├── src
│ └── BuyIt
│    └── Billing
│    ├── Domain
│    │ ├── Model
│    │ └── Service
│    └── Infrastructure
└── tests
```

This style of structure uses an additional subfolder to store the services defined inside the domain model. While this organization may make sense, our preference here is to tend not to use it, since this way of separating code tends to be more focused on the architectural elements rather than the relevant concepts in the model. We believe that this style could easily lead to some sort of service layer on top of the domain model and this is not necessary a bad thing, but keep in mind that Domain Services are used to describe things into the domain, operations that don’t belong to entities nor value objects. So, from now on we will stick with the previous code organization.

这种类型的结构使用一个额外的子文件夹来存储域模型中定义的服务。虽然这个组织可能是有意义的，但是我们在这里倾向于不使用它，因为这种分离代码的方式更倾向于关注架构元素，而不是模型中的相关概念。我们相信,这种风格很容易导致某种服务层上的域模型和必要这并不是一件坏事,但要记住域名服务是用来描述事物到域,操作不属于实体和值对象。因此，从现在开始，我们将继续使用以前的代码组织。

> It’s possible to place code inside the DomainModel subfolder directly. For example, it may be quite common to place common interfaces and services in it, like the DomainEventPub- lisher, the DomainEventSubscriber and so on.
>
> 可以直接将代码放在DomainModel子文件夹中。例如，在其中放置公共接口和服务可能非常常见，比如DomainEventPub- lisher、DomainEventSubscriber等等。

If we had to model a billing context, probably we would have an Order entity with its repository and all the state information. So our first attempt would be to directly place all those elements directly into the DomainModel subfolder. At a first glance, this may seem the simplest way

如果我们必须对帐单上下文建模，那么我们可能会有一个带有其存储库和所有状态信息的Order实体。因此，我们的第一个尝试是将所有这些元素直接放置到DomainModel子文件夹中。乍一看，这似乎是最简单的方法

```
├──  composer.json
├──   composer.lock
├── src
│ └── BuyIt
│    └── Billing
│    ├── DomainModel
│    │ ├── Order.php
│    │ ├── OrderLine.php
│    │ ├── OrderLineWasAdded.php
│    │ ├──  OrderRepository.php
│    │ └── OrderWasCreated.php
│    └── Infrastructure
└── tests
```

We’ve placed the Order and the OrderLine entities, the OrderLineWasAdded and the OrderWasCre- ated event and the OrderRepository into the same subfolder \(DomainModel\). This structure may be fine, but that’s because we still have a simple model. What about the Bill entity plus its repository? Or the Waybill entity plus its respective repository? Let’s add all those elements, and see how it fits into the actual code structure

我们已经将订单和OrderLine实体、OrderLineWasAdded和OrderWasCre事件以及OrderRepository放入相同的子文件夹\(DomainModel\)中。这个结构可能很好，但那是因为我们仍然有一个简单的模型。那么Bill实体及其存储库呢?还是运单实体及其各自的存储库?让我们添加所有这些元素，看看它如何适应实际的代码结构

```
├──  composer.json
├──   composer.lock
├── src
│ └── BuyIt
│    └── Billing
│    ├── DomainModel
│    │ ├── Bill.php
│    │ ├── BillLine.php
│    │ ├── BillLineWasAdded.php
│    │         ├──  BillRepository.php
│    │        ├──  BillWasCreated.php
│    │ ├── Order.php
│    │ ├── OrderLine.php
│    │ ├── OrderLineWasAdded.php
│    │        ├──  OrderRepository.php
│    │      ├── OrderWasCreated.php
│    │ ├── Waybill.php
│    │ ├── WaybillLine.php
│    │ ├── WaybillLineWasAdded.php
│    │ ├── WaybillRepository.php
│    │ └── WaybillWasGenerated.php
│    └── Infrastructure
└── tests
```

While this style of code organization could be fine, it can become non-practical and pretty unmaintainable in the long term. Every time we iterate and add new features, the model will become even more bigger and that subfolder will be eating even more code. We’re in the need to split the code in a way that give us a perspective of the model at a glance. No technical concerns, just domain concerns. To reach this, we can split this model using the Ubiquitous Language, finding meaningful concepts that help us group elements logically in terms of the domain. So we could try an approach as the following

虽然这种类型的代码组织可能很好，但从长远来看，它可能变得不实用，而且非常难以维护。每次我们迭代并添加新特性时，模型就会变得更大，子文件夹就会消耗更多的代码。我们需要以一种能让我们一眼看到模型透视图的方式来分割代码。没有技术问题，只有领域问题。为了达到这个目的，我们可以使用普遍存在的语言来分割这个模型，找到有意义的概念，这些概念可以帮助我们根据域逻辑地对元素进行分组。我们可以试试下面的方法

```
├──  composer.json
├──   composer.lock
├── src
│ └── BuyIt
│    └── Billing
│    ├── DomainModel
│    │ ├── Bill
│    │ │ ├── Bill.php
│    │ │ ├── BillLine.php
│    │ │ ├── BillLineWasAdded.php
│    │ │ ├── BillRepository.php
│    │ │ └── BillWasCreated.php
│    │ ├── Order
│    │ │ ├── Order.php
│    │ │ ├── OrderLine.php
│    │ │ ├── OrderLineWasAdded.php
│    │ │ ├──  OrderRepository.php
│    │ │ └── OrderWasCreated.php
│    │ └── Waybill
│    │    ├── Waybill.php
│    │    ├── WaybillLine.php
│    │    ├── WaybillLineWasAdded.php
│    │    ├── WaybillRepository.php
│    │    └── WaybillWasGenerated.php
│    └── Infrastructure
└── tests
```

This way the code is more organized, conceptually speaking. And not only that. As Evans points out the blue book¹, Modules are a way to communicate as they give us insights about how the domain model works internally, and help us increase the cohesion and decrease the coupling between the concepts. If we look at the previous example, we can see that the concepts Order and OrderLine are strongly related so they live in the same module. On the other hand, Order and Waybill although sharing the same context, they are different concepts so they live in different modules. Modules are not just a way to group related concepts into the model but a way to express part of the design of the model.

从概念上讲，这样代码更有组织。不仅如此。埃文斯指出¹蓝书,模块是一种交流方式,因为他们给我们的见解如何域模型内部工作,并帮助我们增加凝聚力和减少之间的耦合的概念。如果我们看一下前面的例子，我们可以看到Order和OrderLine这两个概念是紧密相关的，因此它们位于同一个模块中。另一方面，订单和运单虽然共享相同的上下文，但它们是不同的概念，因此它们位于不同的模块中。模块不仅是将相关概念分组到模型中的一种方法，而且是表达模型设计的一部分的一种方法。

> Should we place Repositories, Factories, Domain Events, Services in their own subfolder?
>
> Effectively they could be placed into their own subfolder, but it’s strongly discouraged. Just because this way we would be mixing technical concerns and domain concerns, and remember that the Module’s main interest is to group related concepts from the domain model and decouple them from the non-related. Modules don’t separate code but separate meaningful concepts.
>
> 我们应该将存储库、工厂、域事件、服务放在它们自己的子文件夹中吗?
>
> 实际上，它们可以被放到自己的子文件夹中，但这是非常不推荐的。正因为这样，我们将混合技术问题和领域问题，并且记住模块的主要兴趣是将领域模型中的相关概念分组，并将它们与非相关概念解耦。模块不分离代码，而是分离有意义的概念。



