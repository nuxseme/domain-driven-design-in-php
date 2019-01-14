To map the Order entity using the XML mapping, first the setup code of Doctrine should be changed slightly.



```php
require_once "/path/to/vendor/autoload.php";

use Doctrine\ORM\Tools\Setup;
use Doctrine\ORM\EntityManager;

$paths = ["/path/to/mapping-files"];
$isDevMode = false;

// the connection configuration
$dbParams = [
    'driver' => 'pdo_mysql',
    'user' => 'the_database_username', 'password' => 'the_database_password', 'dbname' => 'the_database_name',
];

$config = Setup::createXMLMetadataConfiguration($paths, $isDevMode);
$entityManager  =  EntityManager::create($dbParams,  $config);

```



The mapping file should be created on the path where Doctrine will search for the mapping files. And the mapping files should be named after the fully qualified class name and replacing the backslash \(\\) for dots. So following the example



`Acme\Billing\DomainModel\Order`



Would have the mapping file named as



`Acme.Billing.DomainModel.Order.dcm.xml`



In addition, it’s convenient that all the mapping files use a special XML Schema created specially to specify mapping information



`https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd`



And the final mapping file would be



```php
<?xml version="1.0" encoding="UTF-8"?>
<doctrine-mapping
xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

<entity name="Ddd\Billing\Domain\Model\Order" table="orders">

<id name="id" column="id" type="guid">
<generator strategy="NONE" />
</id>

<field name="amount"
type="decimal" nullable="false" scale="10" precision="5" />

<field name="firstName" type="string" nullable="false" />
<field name="lastName" type="string" nullable="false" />
</entity>

</doctrine-mapping>
```





