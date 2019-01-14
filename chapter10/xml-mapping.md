Doctrine offers multiple formats for the mapping like YAML, XML or annotations. XML is our

preferred choice as it provides robust IDE auto-completion.

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



