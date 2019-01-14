As our Post entity is composed of Value Objects like Body or PostId, it is a good idea to make Custom Mapping Types for them. This will make the object mapping considerably easier.



```php

namespace Infrastructure\Persistence\Doctrine\Types;

use Doctrine\DBAL\Types\Type;
use Doctrine\DBAL\Platforms\AbstractPlatform;
use Domain\Model\Body;

class BodyType extends Type
{
    public function getSQLDeclaration(array $fieldDeclaration, AbstractPlatform $platform)
    {
        return $platform->getVarcharTypeDeclarationSQL($fieldDeclaration);
    }

    /**
    @param string $value
    @return Body
     */
    public function convertToPHPValue($value, AbstractPlatform $platform)
    {
        return new Body($value);
    }

    /**
    @param Body $value
     */
    public function convertToDatabaseValue($value, AbstractPlatform $platform)
    {
        return $value->content();
    }

    public function getName()
    {
        return 'body';
    }
}



namespace Infrastructure\Persistence\Doctrine\Types;

use Doctrine\DBAL\Types\Type;
use Doctrine\DBAL\Platforms\AbstractPlatform;
use Domain\Model\PostId;

class PostIdType extends Type
{
    public function getSQLDeclaration(array $fieldDeclaration, AbstractPlatform $platform)
    {
        return $platform->getGuidTypeDeclarationSQL($fieldDeclaration);
    }

    /**
    @param string $value
    @return PostId
     */
    public function convertToPHPValue($value, AbstractPlatform $platform)
    {
        return new PostId($value);
    }

    /**
    @param PostId $value
     */
    public function convertToDatabaseValue($value, AbstractPlatform $platform)
    {
        return $value->id();
    }

    public function getName()
    {
        return 'post_id';
    }
}

```



Don’t forget to implement the toString magic method to the PostId Value Object, as Doctrine requires this.



```php
class PostId
{

    public function toString()
    {
        return $this->id;
    }
}
```



