Every enterpise application is typically composed of several areas in which the company operates. Areas such as billing, inventory, shipping management, catalog and so on are common examples. The easiest manner in which to manage all these concerns may seem to lean towards a monolithic system. You might wonder, does it have to be this way? What if any friction garnered between teams working on these seperate areas could be reduced by splitting this big monolithic application into smaller, independent chunks. We will now be exploring how to do this, so get prepared for insights and heuristics around strategical design.

每个企业应用程序通常由公司运营的几个领域组成。帐单、库存、运输管理、目录等领域是常见的示例。管理所有这些问题的最简单方式似乎是倾向于单一系统。你可能会想，这是必然的吗?如果在这些独立领域工作的团队之间积累的任何摩擦都可以通过将这个大的单片应用程序分割成更小的、独立的块来减少，会怎么样呢?我们现在将探索如何做到这一点，所以要为战略设计的洞察力和启发式做好准备。

> Dealing With Distributed Systems
>
> Dealing with distributed systems is hard. Breaking a system into independent autonomous
>
> parts has benefits, but it also increases complexity. For example, the coordination and synchronization of those systems is not trivial and as a result should be considered carefully. As Martin Fowler said in the PoEAA book, the first law of distributed systems is always: Don’t distribute.
>
> 处理分布式系统
>
> 处理分布式系统是困难的。把一个系统分解成独立自主的
>
> 部件有好处，但也增加了复杂性。例如，这些系统的协调和同步不是微不足道的，因此应该仔细考虑。正如Martin Fowler在PoEAA的书中所说，分布式系统的第一定律总是:不要分发。



