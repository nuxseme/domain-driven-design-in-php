Data Access Objects are a common pattern for persisting domain objects into the database. It is easy to confuse the Data Access Object pattern with a Repository. The significant difference being that Repositories represent collections, whilst DAOs are closer to the database, often being far more table-centric. Typically a DAO would contain CRUD methods for a particular domain object.

A common interface for a DAO could be



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



