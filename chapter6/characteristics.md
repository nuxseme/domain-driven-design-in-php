Domain events are ordinarily immutable, as they are a record of something in the past. In addition to a description of the event, a domain event typically contains a timestamp for the time the event occurred and the identity of entities involved in the event. Also, a domain event often has a separate timestamp indicating when the event was entered into the system and the identity of the person who entered it. When useful, an identity for the domain event can be based on some set of these properties. So, for example, if two instances of the same event arrive at a node they can be recognized as the same.

域事件通常是不可变的，因为它们是过去事物的记录。除了事件的描述之外，域事件通常还包含事件发生的时间戳和事件中涉及的实体的标识。此外，域事件通常具有独立的时间戳，指示事件何时进入系统以及进入该事件的人的身份。如果有用，域事件的标识可以基于这些属性的某个集合。例如，如果同一个事件的两个实例到达一个节点，它们可以被识别为相同的实例。

The essence of a Domain Event is that you use it to capture things that can trigger a change to the state of the application you are developing or to another applications in your Domain interested in those changes. These event objects are then processed to cause changes to the system, and stored to provide an audit log.

域事件的本质是，您可以使用它来捕获可以触发对正在开发的应用程序状态的更改的事件，或对这些更改感兴趣的域内其他应用程序的更改。然后处理这些事件对象以引起对系统的更改，并存储它们以提供审计日志。

