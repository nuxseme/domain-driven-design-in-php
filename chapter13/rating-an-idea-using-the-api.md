During the day, your Product Owner comes to you and says: “by the way, a user should be able to rate an idea using our mobile app. I think we will need to update the API, could you do it for this sprint?”. Here’s the PO again. “No problem!”. Business is impressed with your commitment.

As Robert C. Martin says: “The Web is a delivery mechanism \[…\] Your system architecture should be as ignorant as possible about how it is to be delivered. You should be able to deliver it as a console app, a web app, or even a web service app, without undue complication or any change to the fundamental architecture”.

Your current API is built using Silex, the PHP micro-framework based on the Symfony2 Compo- nents. Let’s go for it in Listing





```php
require_once __DIR__.'/../vendor/autoload.php';
$app = new \Silex\Application();
// ... more routes
$app->get(
    '/api/rate/idea/{ideaId}/rating/{rating}',
    function($ideaId, $rating) use ($app) {
        $ideaRepository = new RedisIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );
        return $app->json($response->idea);
    }
);
$app->run();
```





Is there anything familiar to you? Can you identify some code that you have seen before? I’ll give you a clue.





```php
$ideaRepository = new RedisIdeaRepository();
$useCase = new RateIdeaUseCase($ideaRepository);
$response = $useCase->execute(
    new RateIdeaRequest($ideaId, $rating)
);
```



Man! I remember those 3 lines of code. They look exactly the same as the web application. That’s right, because the UseCase encapsulates the business rules you need to prepare the request, get the response and act accordingly.

We are providing our users with another way for rating an idea; another delivery mechanism.

The main difference is where we created the RateIdeaRequest from. In the first example, it was from a ZF request and now it is from a Silex request using the parameters matched in the route.



