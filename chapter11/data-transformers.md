There are some cases where generating intermediate DTOs for more complex responses like JSON, XML, CSV, iCAL Contact could be seen as an unnecessary overhead. We could output the representation in a buffer and ask for it later on in the delivery side.

Transformers transform high-level domain concepts into low-level client details. Let’s see an example

在某些情况下，为JSON、XML、CSV、iCAL Contact等更复杂的响应生成中间dto可能被视为不必要的开销。我们可以将表示形式输出到缓冲区中，稍后在交付端请求它。



转换器将高级域概念转换为低级客户端细节。让我们看一个例子

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

考虑为给定产品生成不同数据表示的情况。通常产品信息是通过web接口\(HTML\)提供的，但我们可能对提供其他格式\(如XML、JSON或CSV\)感兴趣。这可能支持与其他服务的集成。



博客也是类似的情况。我们可能会向世界展示我们作为HTML作者的潜力，但有些人会对通过RSS阅读我们的文章感兴趣。用例应用程序服务保持不变，表示则不同。



DTO是一种干净而简单的解决方案，可以传递给模板引擎用于不同的表示，但是它可能会使数据转换的最后一步的逻辑变得复杂。这些模板的逻辑可能成为维护、测试和理解的问题。



对于特定的情况，数据转换器可能是更好的方法。这些只是一些黑盒，域概念作为输入\(聚合、实体等\)，只读表示\(XML、JSON、CSV等\)作为输出。这个变形器很容易测试。

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

这很容易。想知道XML或CSV是什么样子的吗?



让我们看看如何将数据转换器与我们的应用程序服务集成起来。

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

类似于DTO汇编方法，但这次不返回具体值。数据转换器用于保存和与数据交互。



DTO的主要问题是编写它们的开销。大多数情况下，域概念和DTO表示将呈现相同的结构。大多数时候你会觉得不值得。



> 当您的表示层中的模型与底层域模型之间存在严重的不匹配时，使用DTO之类的东西是有用的。在这种情况下，将特定于表示的门面/网关映射到域模型并提供便于表示的接口是有意义的。它非常适合表示模型。这是值得做的，但它只值得做的屏幕有这种不匹配\(在这种情况下，这不是额外的工作，因为你必须在屏幕上做。

我们认为长远的眼光是值得投资的。在中型到大型项目中，接口表示和领域概念以非常不同的节奏变化。您可能希望将它们彼此解耦以降低更新的摩擦。使用DTO或数据转换器允许您自由地发展您的模型，而不必一直考虑破坏布局。

