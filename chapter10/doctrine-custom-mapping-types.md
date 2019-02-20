As our Post entity is composed of Value Objects like Body or PostId, it is a good idea to make Custom Mapping Types for them. This will make the object mapping considerably easier.

由于我们的Post实体由诸如Body或PostId之类的值对象组成，因此为它们定制映射类型是个好主意。这将大大简化对象映射。

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

不要忘记对PostId值对象实现toString魔法方法，因为Doctrine需要这样做。

```php
class PostId
{

    public function toString()
    {
        return $this->id;
    }
}
```



