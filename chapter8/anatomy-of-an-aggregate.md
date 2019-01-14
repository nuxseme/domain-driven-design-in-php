An Aggregate is conformed by a root Entity that holds other Entities and Value Objects. A single Entity without any child Entities or Value Objects conforms an Aggregate by itself. That’s why in some books the term Aggregates is used over the term Entity.

The main benefit or their real goal of an Aggregate is consistency in our Domain Model operations. It allows us to guarantee that changes on a hierarchy of Entities and Value Objects are performed atomically.

Consider a rough examples to introduce the idea. Imagine an e-commerce application and a typical persistence mechanism. There are Orders and Line Orders. Orders total amounts and Line Orders subtotal amounts sum must match. You will never perform changes in a Line Order amount and in the Order amount without using a transaction protecting both changes. Why? Because the UPDATE statement that updates the Line Order can work properly while the UPDATE on the Order total amount could fail due to network connectivity issues. With such situation, you would end up with a inconsistency in tables and your Domain, what would happen if you ask for the total amount in the Order and then you apply the business logic to calculate the sum of all the Line Orders? You get the idea. If you put both changes in a transaction, both will succeed or both will fail, that’s consistency. So transactions help you keep this consistency.

On the other side, the negative effects are performance issues and persistence errors.

An Aggregate is fetched and persisted using its own repository, whether it holds many Entities and Value Objects or none.

In order to design an Aggregate, there are some rules or considerations to follow so we can get all the benefits minimizing the negative effects.



