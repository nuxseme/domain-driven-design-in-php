Doctrine offers multiple formats for the mapping like YAML, XML or annotations. XML is our

preferred choice as it provides robust IDE auto-completion.

Doctrine为映射提供多种格式，如YAML、XML或注释。XML是我们

首选选项，因为它提供了健壮的IDE自动完成功能。

```php
<?xml version="1.0" encoding="UTF-8"?>
<doctrine-mapping
xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
http://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">
<entity name="Domain\Model\Post" table="posts">
<id name="id" type="post_id" column="id">
<generator strategy="NONE" />
</id>
<field name="body" type="body" length="250" column="body"/>
<field name="createdAt" type="datetime" column="created_at" />
</entity>
</doctrine-mapping>
```



