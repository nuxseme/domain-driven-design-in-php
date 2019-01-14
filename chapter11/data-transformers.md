There are some cases where generating intermediate DTOs for more complex responses like JSON, XML, CSV, iCAL Contact could be seen as an unnecessary overhead. We could output the representation in a buffer and ask for it later on in the delivery side.

Transformers transform high-level domain concepts into low-level client details. Let’s see an example



```php
interface UserDataTransformer
{
    public function write(User $user);
    /**
     * @return mixed
     */
    public function read();
}
```



Consider the case of generating different data representations for a given product. Usually the product information is served through a web interface \(HTML\) but we might be interested in offer other formats like XML, JSON or CSV. This might enable integrations with other services.

Similar case for a Blog. We might expose to the world our potential as writers in HTML but some people will be interested in consuming our articles through RSS. The use cases Application Services remain the same, the representation does not.

DTO’s are a clean and simple solution that could be passed to template engines for different representations but it might complicate the logic of this last step of data transformation. The logic for such templates could become a problem to maintain, test and understand.

Data Transformers might be a better approach on specific cases. These are just black boxes with Domain Concepts as inputs \(Aggregates, Entities, etc.\) and read-only representations \(XML, JSON, CSV, etc.\) as outputs. This transformers could be really easy to test.

```php
class JsonUserDataTransformer implements UserDataTransformer
{
    private $data;
    public function write(User $user)
    {
    // More complex logic could be placed here
    // As using JMSSerializer, native json, etc.
        $this->data = json_encode($user);
    }
    /**
     * @return string
     */
    public function read()
    {
        return $this->data;
    }
}
```





That was easy. Wondering how would look like the XML or CSV one?

Let’s see how to integrate the Data Transformer with our Application Service.



```php

class SignUpUserService
{
    private $userRepository;
    private $userDataTransformer;
    public function __construct(
        UserRepository $userRepository,
        UserDataTransformer $userDataTransformer
    )
    {
        $this->userRepository = $userRepository;
        $this->userDataTransformer = $userDataTransformer;
    }
    public function execute(SignUpUserRequest $request)
    {
        // ...
        $user = // ...
        $this->userDataTransformer->write($user);
    }
    /**
     * @return UserDataTransformer
     */
    public function userDataTransformer()
    {
        return $this->userDataTransformer;
    }
}
```



Similar to the DTO Assembler approach but this time without returning a concrete value. The Data Transfomer is being used to hold and interact with the data.

The main issue with DTO’s is the overhead of writing them. Most of the time your Domain concepts and DTO representations will present the same structure. Most of the time you’ll feel is not worthy.



> One case where it is useful to use something like a DTO is when you have a significant mismatch between the model in your presentation layer  and  the  underlying  domain model. In this case it makes sense to make presentation specific facade/gateway that maps from the domain model and presents an interface that’s convenient for the presentation. It fits in nicely with Presentation Model. This is worth doing, but it’s only worth doing for screens that have this mismatch \(in this case it isn’t extra work, since you’d have to do it in the screen anyway.\) – Martin Fowler, PoEAA



We think the long-term vision will worth the investment. On medium to big projects, interface representations and Domain concepts change at very different rhythms. You might want to decouple them from each other to lower the friction for updates. Using DTO’s or Data Transformers allow you to evolve your model freely without having to think about breaking the layout all the time.



