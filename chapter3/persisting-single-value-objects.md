Many different options are available to persist a single Value Object. These range from using Serialize LOB or Embedded Value as mapping strategies, to use an ad-hoc ORM or an open-source alternative, such as Doctrine. We consider an ad-hoc ORM to be a custom built ORM that your company may have developed in order to persist Entities in a database. In our scenario, the ad-hoc ORM code is going to be implemented using the DBAL¹¹ library. The Doctrine database abstraction and access layer \(DBAL\) offers a lightweight runtime around a PDO-like API, along with additional features such as, database schema introspection and manipulation through an OO API.



