With modern RPC we refer to RPC through RESTful resources. A Bounded Context reveals to the outside world a clear interface to interact with. It exposes resources that could be manipulated through HTTP verbs. We could say that the Bounded Context offers a set of services and operations. In strategical terms, this is what is called an Open Host Service.

在现代RPC中，我们通过RESTful资源引用RPC。有界上下文向外部世界展示了一个可交互的清晰接口。它公开了可以通过HTTP谓词操作的资源。我们可以说，有界上下文提供了一组服务和操作。从战略上讲，这就是所谓的开放主机服务。

> Open Host Service
>
> Define a protocol that gives access to your subsystem as a set of SERVICES. Open the protocol
>
> so that all who need to integrate with you can use it. Enhance and expand the protocol to handle new integration requirements, except when a single team has idiosyncratic needs. Then, use a one-off translator to augment the protocol for that special case so that the shared protocol can stay simple and coherent.
>
> Eric Evans - Domain-Driven Design, Tackling complexity in the heart of software.
>
> 打开主机服务
>
>
>
> 定义一个协议，该协议将对子系统的访问作为一组服务进行访问。开放的协议
>
>
>
> 这样所有需要集成的人都可以使用它。增强和扩展协议以处理新的集成需求，除非单个团队有特殊的需求。然后，使用一次性的转换器来为这种特殊情况增强协议，以便共享协议能够保持简单和一致。
>
>
>
> 领域驱动的设计，解决软件核心的复杂性。

Lets explore an example provided within the Last Wishes application¹ that comes with the books’ Github organization.

The application is a web platform whose purpose is letting people save their last wills before they die. There are two contexts, one responsible in handling wills – the Will Bounded Context – and the Gamification Context² in charge of giving points to the users of the system. In the Will Context, the user could have badges that are related to the number of points the user made on the Gamification Context. This means that we need to integrate both contexts together in order to show the badges a user has on the Will Context.

让探索遗愿中提供一个例子程序¹跟书的Github组织。



该应用程序是一个web平台，其目的是让人们在死前保存自己的遗嘱。有两个情况下,一个负责处理遗嘱——将限界上下文和游戏化上下文²负责给系统的用户点。在Will上下文中，用户可以拥有与用户在游戏化上下文中获得的点数相关的徽章。这意味着我们需要将这两个上下文集成在一起，以显示用户在Will上下文中拥有的徽章。

The Gamification Context is a full-fledged event-driven application powered by a custom eventsourc- ing engine. It is a full-stack Symfony application that uses FOSRestBundle³, BazingaHateoasBun- dle⁴, JMSSerializerBundle⁵, NelmioApiDocBundle⁶ and OngrElasticsearchBundle⁷ to provide a level 3 and up REST API \(commonly known as the Glory of REST \) according to the Richardson Maturity Model⁸. All the events triggered within this Context are projected against an Elasticsearch server in order to produce the data needed for the views. We will expose the number of points made  for a given user through an endpoint like [http://gamification.context.host/api/users/{id}](http://gamification.context.host/api/users/{id}). We will fetch the user projection from Elasticsearch and serialise it to a format previously negotiated with the client.

游戏化上下文是一个成熟的事件驱动应用程序，由定制的eventsourc- ing引擎提供支持。它是一个完整的Symfony应用程序,它使用FOSRestBundle³,BazingaHateoasBun——⁴,JMSSerializerBundle⁵,NelmioApiDocBundle⁶和OngrElasticsearchBundle⁷来提供一个3级和REST API\(俗称的荣耀REST\)根据理查森⁸成熟度模型。在这个上下文中触发的所有事件都投射到Elasticsearch服务器上，以生成视图所需的数据。我们将通过诸如http://gamific.context.host/api/users/{id}这样的端点公开为给定用户所做的点数。我们将从Elasticsearch获取用户投影，并将其序列化为之前与客户协商的格式。

```php
namespace AppBundle\Controller;
use FOS\RestBundle\Controller\Annotations as Rest;
use FOS\RestBundle\Controller\FOSRestController;
use Nelmio\ApiDocBundle\Annotation\ApiDoc;
class UsersController extends FOSRestController
{
    /**
     * @ApiDoc(
     * resource=true,
     * description="Finds a user given a user ID",
     * statusCodes={
     * 200 = "Returned when the user have been found",
     * 404 = "Returned when the user could not be found"
     * }
     * )
     *
     * @Rest\View(
     * statusCode = 200
     * )
     */
    public function getUserAction($id)
    {
        $repo = $this->get('es.manager.default.user');
        $user = $repo->find($id);
        if (!$user) {
            throw $this->createNotFoundException(
                sprintf(
                    'A user with an ID of %s does not exist',
                    $id
                )
            );
        }
        return $user;
    }
}
```

As we explained in the architecture chapter, reads are treated as an infrastructure concern. There is no need to wrap them inside a Command / Command Handler flow.

The resulting JSON+HAL representation of a user will be

正如我们在架构一章中所解释的，读取被视为一个基础架构问题。不需要将它们包装在命令/命令处理程序流中。

用户的JSON+HAL表示形式将是

```php
{
    "id": "c3c587c6-610a-42df-90d3-8e9a181d65d0",
    "points": 0,
    "_links": {
        "self": {
            "href": "http://gamification.context/api/users/c3c587c6-610a-42df-90d3-8e9a181d65d0"
        }
    }
}
```

Now we are in a good position for integrating both contexts. We just need to write the client in the Will Context for consuming the endpoint we have just created. Should we mix both domain models? Digesting the Gamification Context directly will mean adapting the Will Context to the Gamification one, resulting in a Conformist integration. However, separating these concerns seems worth the investment. We need a layer for guaranteeing the integrity and the consistency of the Domain Model within the Will Context. We need to translate points \(Gamification\) to badges \(Will\). This translation mechanism is what in DDD is called an Anti-corruption Layer.

现在我们可以很好地集成这两个上下文。我们只需要在Will上下文中编写客户机来消费我们刚刚创建的端点。我们应该混合这两个领域模型吗?直接消化游戏化上下文意味着将意志上下文与游戏化上下文相适应，从而形成一种墨守成规的集成。然而，分离这些担忧似乎值得投资。我们需要一个层来保证Will上下文中域模型的完整性和一致性。我们需要将分数\(游戏化\)转化为徽章\(意愿\)。这种翻译机制在DDD中被称为反腐败层。

> Anti-corruption Layer

Create an isolating layer to provide clients with functionality in terms of their own domain

model. The layer talks to the other system through its existing interface, requiring little or no modification to the other system. Internally, the layer translates in both directions as necessary between the two models.

Eric Evans - Domain-Driven Design, Tackling complexity in the heart of software.

So, what does the Anti-corruption layer look like? Most of the time Service will be interacting with a combination of Adapters and Facades. The Services encapsulate and hide the complexities behind these complex transformations. Facades aid in hiding and encapsulating access details required in fetching data from the Gamification model. Adapters translate between models, often using specialised Translators.

Lets see how to define a User Service within the Will’s model, that will be responsible to retrieve the badges earned by a given user.

创建一个隔离层，根据客户自己的域为其提供功能



模型。该层通过其现有的接口与另一个系统进行通信，只需要对另一个系统进行很少或根本不需要修改。在内部，层根据需要在两个模型之间进行双向转换。



领域驱动的设计，解决软件核心的复杂性。



那么，反腐败层是什么样的呢?大多数时候，服务将与适配器和外观的组合进行交互。服务封装并隐藏这些复杂转换背后的复杂性。立面有助于隐藏和封装从游戏化模型中获取数据所需的访问细节。适配器在模型之间转换，通常使用专门的翻译器。



让我们看看如何在Will的模型中定义一个用户服务，该服务将负责检索给定用户获得的徽章。

```php
namespace Lw\Domain\Model\User;
interface UserService
{
    public function badgesFrom(UserId $id);
}
```

Now, the implementation in the Infrastructure side. We will use an adapter for the transformation process.

现在，基础设施方面的实现。我们将为转换过程使用适配器。

```php
namespace Lw\Infrastructure\Service;
use Lw\Domain\Model\User\UserId;
use Lw\Domain\Model\User\UserService;
class TranslatingUserService implements UserService
{
    private $userAdapter;
    public function __construct(UserAdapter $userAdapter)
    {
        $this->userAdapter = $userAdapter;
    }
    public function badgesFrom(UserId $id)
    {
        return $this->userAdapter->toBadges($id);
    }
}
```

The Adapter for the transformation

转换的适配器

```php
namespace Lw\Infrastructure\Service;
use GuzzleHttp\Client;
class HttpUserAdapter implements UserAdapter
{
    private $client;
    public function __construct(Client $client)
    {

        $this->client = $client;
    }
    public function toBadges($id)
    {
        $response = $this->client->get(sprintf('/users/%s', $id), [
            'allow_redirects' => true,
            'headers' => [
                'Accept' => 'application/hal+json'
            ]
        ]);
        $badges = [];
        if (200 === $response->getStatusCode()) {
            $badges =
                (new UserTranslator())
                    ->toBadgesFromRepresentation(
                        json_decode(
                            $response->getBody(),
                            true
                        )
                    )
            ;
        }
        return $badges;
    }
}
```

As you can see, the Adapter acts as a Facade to the Gamification Context to. We did it this way as fetching the User resource in the Gamification side is pretty straightforward. The Adapter uses the UserTranslator to perform the translation.

如您所见，适配器充当游戏化上下文的Facade。我们这样做是因为在游戏化方面获取用户资源非常简单。适配器使用UserTranslator执行转换。

```php
namespace Lw\Infrastructure\Service;
use Lw\Infrastructure\Domain\Model\User\FirstWillMadeBadge;
use Symfony\Component\PropertyAccess\PropertyAccess;
class UserTranslator
{
    public function toBadgesFromRepresentation($representation)
    {
        $accessor = PropertyAccess::createPropertyAccessor();
        $points = $accessor->getValue($representation, 'points');
        $badges = [];
        if ($points > 3) {
            $badges[] = new FirstWillMadeBadge();
        }
        return $badges;
    }
}
```

The Translator specialises in transforming the points coming from the Gamification Context into badges.

We have shown how to integrate two Bounded Contexts where respective teams share a Customer

/ Supplier relationship. The Gamification Context exposes the integration through an Open Host Service implemented by a RESTful protocol. On the other side, the Will Context consumes the service through an Anti-corruption Layer responsible in translating the model from one domain to the other, ensuring the Will Contexts’ integrity.

翻译的专长是将游戏化背景下的点数转换成徽章。



我们已经展示了如何集成两个边界上下文，其中各个团队共享一个客户



/供应商关系。游戏化上下文通过rest协议实现的开放主机服务公开集成。另一方面，Will上下文通过负责将模型从一个域转换到另一个域的反腐败层使用服务，从而确保Will上下文的完整性。

