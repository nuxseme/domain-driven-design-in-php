Think about a JavaScript 2D platform game. There are tons of different components interacting with each other on the screen at the same time. There is a component that indicates the number of lives remaining, another one that shows all the points scored, or another one counting down the time remaining to finish the current level. Each time the player jumps on an enemy your points scored get increased. When your scoring goes higher than a certain number of points, you get an extra life. When a player collides against a key, it gets captured and probably a door opens. How all these components interact with each other? What’s the optimal architecture for this scenario?

There are probably two main options: the first one is to couple each component with the ones it is connected to. In the example, the player components would be coupled with too many other components probably. When a new component is added to the game, the developer needs to modify the code of the first one. Do you remember the Open-Closed principle¹? Adding a new component shouldn’t make the first component to be updated. What would happen with too many components? Is it easy to maintain? Not at all.

The second approach is connecting all the components to a single object that handles all the important events in the game. It receives events from each component and it forwards them to specific components. For example, the scoring component would be interested in an EnemyKilled event, or the LifeCaptured event is quite useful for the player entity and the remaining lives component. In this way, all components are coupled to a single component that manages all the notifications. With this approach, adding new components or removing existing ones do not affect the remaining ones.

While developing a single application, events come handy to decoupling components. When developing a whole domain in a distributed way, events are very useful to decouple each service or application that plays a role in the domain. The key points are the same but at a different scale.



考虑一个JavaScript 2D平台游戏。屏幕上同时有大量不同的组件相互交互。有一个组件指示剩余生命的数量，另一个组件显示所有得分，或者另一个组件计算剩余时间以完成当前级别。每次玩家跳到敌人身上，你的得分就会增加。当你的分数高于一定的分数时，你会得到额外的生命。当玩家与钥匙发生碰撞时，钥匙就会被捕获并打开一扇门。所有这些组件如何相互交互?这个场景的最佳架构是什么?

可能有两个主要选项:第一个是将每个组件与其所连接的组件耦合起来。在本例中，播放器组件可能与太多其他组件耦合。当向游戏中添加新组件时，开发人员需要修改第一个组件的代码。你还记得开放闭合原则¹吗?添加新组件不应该使第一个组件被更新。如果有太多的组件会发生什么?它容易维护吗?不客气。

第二种方法是将所有组件连接到一个处理游戏中所有重要事件的对象。它从每个组件接收事件，并将它们转发到特定的组件。例如，记分组件可能对enemykill事件感兴趣，或者生命周期事件对玩家实体和剩余生命组件非常有用。通过这种方式，所有组件都耦合到管理所有通知的单个组件。使用这种方法，添加新组件或删除现有组件不会影响其余组件。

在开发单个应用程序时，事件可以方便地解耦组件。当以分布式方式开发整个域时，事件对于解耦域中扮演角色的每个服务或应用程序非常有用。要点是相同的，但规模不同。

