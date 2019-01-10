Domain events are ordinarily immutable, as they are a record of something in the past. In addition to a description of the event, a domain event typically contains a timestamp for the time the event occurred and the identity of entities involved in the event. Also, a domain event often has a separate timestamp indicating when the event was entered into the system and the identity of the person who entered it. When useful, an identity for the domain event can be based on some set of these properties. So, for example, if two instances of the same event arrive at a node they can be recognized as the same.

The essence of a Domain Event is that you use it to capture things that can trigger a change to the state of the application you are developing or to another applications in your Domain interested in those changes. These event objects are then processed to cause changes to the system, and stored to provide an audit log.



