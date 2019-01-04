As there are plenty of options for architecture styles you could get a bit confused in this chapter. You will have to consider the trade-offs for each one of them in order to choose wisely. One thing is clear, the Big Ball of Mud approach is not an option as the code will rot pretty fast. Layered architecture is a better option but it presents some disadvantages like tight coupling between layers. Arguably, the most balanced option would be the Hexagonal Architecture, as it can be used as a foundational base architecture. It promotes a high-level degree of decoupling and symmetry between the inside and outside of the application.

We have also seen CQRS and Event Sourcing as pretty flexible architectures that will help you in fighting serious complexity. CQRS and Event Sourcing have their place but do not let ‘the coolness factor’ distract you from the value they provide. As they come with some overheads, you should have a technical reason for justifying its use. These architectural styles are indeed really useful and the heuristics to start using them can be discovered in the number of finders on the repositories for CQRS and the volume of triggered events for Event Sourcing. If the number of finder methods starts growing and repositories become hard to maintain then it is time to consider the use of CQRS, to split read and write concerns. And after that, if the volume of events on each aggregate operation tends to grow and the business is interested in more granular information then an option to consider is whether a move towards Event Sourcing would pay off.



由于体系结构样式有很多选择，所以在本章中您可能会感到有些困惑。为了做出明智的选择，你必须考虑每一种选择的利弊。有一件事是清楚的，大泥球方法不是一个选项，因为代码会很快腐烂。分层架构是更好的选择，但它也存在一些缺点，比如层之间的紧密耦合。可以说，最平衡的选择是六角形架构，因为它可以用作基础架构。它促进了应用程序内外的高级解耦和对称。



我们还将CQRS和事件源视为非常灵活的体系结构，可以帮助您克服严重的复杂性。CQRS和活动来源有它们的位置，但不要让“酷元素”分散你对它们提供的价值的注意力。由于它们带有一些开销，您应该有一个技术上的理由来证明它的使用。这些体系结构风格确实非常有用，从CQRS存储库上的查找器数量和事件源触发事件的数量中可以发现开始使用它们的启发式方法。如果查找器方法的数量开始增长，并且存储库变得难以维护，那么就应该考虑使用CQRS，以分割读和写关注点。在此之后，如果每个聚合操作上的事件量趋于增长，并且业务对更细粒度的信息感兴趣，那么可以考虑的一个选项是，向事件源的转变是否会带来回报



---

...

