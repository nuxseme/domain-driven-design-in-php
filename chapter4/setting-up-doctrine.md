First of all we need to require it through composer. In the root of the project the command below has to be executed



`> php composer.phar require "doctrine/orm=~2.4"`



And then these lines will allow to setup doctrine

```php
require_once "/path/to/vendor/autoload.php";

use Doctrine\ORM\Tools\Setup;
use Doctrine\ORM\EntityManager;

$paths = ["/path/to/entity-files"];
$isDevMode = false;

// the connection configuration
$dbParams = array(
    'driver' => 'pdo_mysql',
    'user' => 'the_database_username', 'password' => 'the_database_password', 'dbname' => 'the_database_name',
);

$config = Setup::createAnnotationMetadataConfiguration($paths, $isDevMode);
$entityManager  =  EntityManager::create($dbParams,  $config);
```



