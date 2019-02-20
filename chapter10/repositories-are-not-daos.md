Data Access Objects are a common pattern for persisting domain objects into the database. It is easy to confuse the Data Access Object pattern with a Repository. The significant difference being that Repositories represent collections, whilst DAOs are closer to the database, often being far more table-centric. Typically a DAO would contain CRUD methods for a particular domain object.

数据访问对象是将域对象持久化到数据库中的常见模式。很容易将数据访问对象模式与存储库混淆。显著的区别在于存储库表示集合，而dao更接近数据库，通常以表为中心。通常，DAO将包含特定域对象的CRUD方法。

A common interface for a DAO could be

DAO的公共接口可以是

```php
interface UserDAO
{
    /**
    @param string $username
    @return User
     */
    public function get($username); 
    public function create(User $user); 
    public function update(User $user);
    /**
    @param string $username
     */
    public function delete($username);
}
```

A DAO interface could have multiple implementations which could range from ORM constructions to plain SQL queries.

The main problem with DAOs is that their responsibilities are not clearly defined. DAOs are usually perceived as a gateway to the database so it is pretty easy to greatly decrease cohesion with many specific methods to query the database.

DAO接口可以有多种实现，从ORM构造到普通SQL查询。

dao的主要问题是它们的职责没有明确定义。dao通常被视为数据库的网关，因此很容易与许多查询数据库的特定方法大大降低内聚性。

```php
interface BloatUserDAO
{
    public function get($username); 
    public function create(User $user); 
    public function update(User $user); 
    public function delete($username);
    public function getUserByLastName($lastName); 
    public function getUserByEmail($email);
    public function updateEmailAddress($username, $email);

    public function updateLastName($username, $lastName);
}
```

As you see, the DAO becomes harder to unit test as you need to implement more methods and it becomes more coupled to the User object. This problem will grow over-time, with many other contributors collaborating in making the ball of mud even bigger.

如您所见，DAO变得更难进行单元测试，因为您需要实现更多的方法，而且它与用户对象的耦合程度更高。随着时间的推移，随着许多其他贡献者的合作，这个问题会越来越严重。

