In order to interact with a domain object you need to hold a reference to it. One way of achieving this is by creation, alternatively you can traverse an association.

In Object-Oriented programs, objects have links \(references\) to other objects, which make them easily traversable. This contributes greatly to our models expressive power. The catch being, that you need a mechanism to retrieve the first object, the Aggregate Root.

Repositories act as storage locations, where a retrieved object is returned in the exact same state it was persisted in - making them very easy to reason about.

Every Aggregate type typically has a unique associated Repository, used for its persistence needs. In the case however, where it is required to share an Aggregate object hierarchy, the types might share a repository.

Once you have successfully retrieved the Aggregate from the repository, every change you make is persisted. Removing the need to go back to the repository.



