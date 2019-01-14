Is it normal that I have to create so many dependencies by hand? No. It’s common to use a Dependency Injection component or a Service Container with such capabilities. Again, Symfony comes to the rescue, however, you can also check PHP-DI 4 http://php-di.org/.

Let’s see the resulting code in Listing 14 after applying Symfony Service Container component to our application.





```php
class IdeaController extends ContainerAwareController
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');
        $useCase = $this->get('rate_idea_use_case');
        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );
        $this->redirect('/idea/'.$response->idea->getId());
    }
}
```



The controller has been modified to have access to the container, that’s why it is inheriting from a new base controller ContainerAwareController that has a get method to retrieve each of the services contained.





```php
<?xml version="1.0" ?>
<container xmlns="http://symfony.com/schema/dic/services"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://symfony.com/schema/dic/services
http://symfony.com/schema/dic/services/services-1.0.xsd">
    <services>
        <service
                id="rate_idea_use_case"
                class="RateIdeaUseCase">
            <argument type="service" id="idea_repository" />
        </service>
        <service
                id="idea_repository"
                class="RedisIdeaRepository">
            <argument type="service">
                <service class="Predis\Client" />
            </argument>
        </service>
    </services>
</container>
```





In Listing, you can also find the XML file used to configure the Service Container. It’s really easy to understand but if you need more information, take a look to the Symfony Service Container Component site in http://symfony.com/doc/current/book/service\_container.html



