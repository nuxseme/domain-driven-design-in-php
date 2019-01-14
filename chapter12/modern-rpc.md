With modern RPC we refer to RPC through RESTful resources. A Bounded Context reveals to the outside world a clear interface to interact with. It exposes resources that could be manipulated through HTTP verbs. We could say that the Bounded Context offers a set of services and operations. In strategical terms, this is what is called an Open Host Service.



> Open Host Service
>
> Define a protocol that gives access to your subsystem as a set of SERVICES. Open the protocol
>
> so that all who need to integrate with you can use it. Enhance and expand the protocol to handle new integration requirements, except when a single team has idiosyncratic needs. Then, use a one-off translator to augment the protocol for that special case so that the shared protocol can stay simple and coherent.
>
> Eric Evans - Domain-Driven Design, Tackling complexity in the heart of software.



Lets explore an example provided within the Last Wishes application¹ that comes with the books’ Github organization.

The application is a web platform whose purpose is letting people save their last wills before they die. There are two contexts, one responsible in handling wills – the Will Bounded Context – and the Gamification Context² in charge of giving points to the users of the system. In the Will Context, the user could have badges that are related to the number of points the user made on the Gamification Context. This means that we need to integrate both contexts together in order to show the badges a user has on the Will Context.

The Gamification Context is a full-fledged event-driven application powered by a custom eventsourc- ing engine. It is a full-stack Symfony application that uses FOSRestBundle³, BazingaHateoasBun- dle⁴, JMSSerializerBundle⁵, NelmioApiDocBundle⁶ and OngrElasticsearchBundle⁷ to provide a level 3 and up REST API \(commonly known as the Glory of REST \) according to the Richardson Maturity Model⁸. All the events triggered within this Context are projected against an Elasticsearch server in order to produce the data needed for the views. We will expose the number of points made  for a given user through an endpoint like http://gamification.context.host/api/users/{id}. We will fetch the user projection from Elasticsearch and serialise it to a format previously negotiated with the client.





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



     Anti-corruption Layer

Create an isolating layer to provide clients with functionality in terms of their own domain

model. The layer talks to the other system through its existing interface, requiring little or no modification to the other system. Internally, the layer translates in both directions as necessary between the two models.

Eric Evans - Domain-Driven Design, Tackling complexity in the heart of software.



So, what does the Anti-corruption layer look like? Most of the time Service will be interacting with a combination of Adapters and Facades. The Services encapsulate and hide the complexities behind these complex transformations. Facades aid in hiding and encapsulating access details required in fetching data from the Gamification model. Adapters translate between models, often using specialised Translators.

Lets see how to define a User Service within the Will’s model, that will be responsible to retrieve the badges earned by a given user.



```php
namespace Lw\Domain\Model\User;
interface UserService
{
    public function badgesFrom(UserId $id);
}
```





Now, the implementation in the Infrastructure side. We will use an adapter for the transformation process.



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





