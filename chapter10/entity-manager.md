The EntityManager is the central access point for the ORM functionality, bootstraping it is easy as

EntityManager是ORM功能的中心访问点，引导非常简单

```php
use Doctrine\DBAL\Types\Type;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Tools;

Type::addType('post_id',  'Infrastructure\Persistence\Doctrine\Types\PostIdType');
Type::addType('body',  'Infrastructure\Persistence\Doctrine\Types\BodyType');

$entityManager = EntityManager::create( [
    'driver' => 'pdo_sqlite',
    'path' =>   DIR     . '/db.sqlite',
],
    Tools\Setup::createXMLMetadataConfiguration( ['/Path/To/Infrastructure/Persistence/Doctrine/Mapping'],
        $devMode = true
    )
);
```

Remember to configure as per your needs and setup.

请记住根据您的需要和设置进行配置。

