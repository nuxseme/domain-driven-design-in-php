In the case of needing to persist an Entity with a collection of Value Objects and need querying capabilities, you have the choice to persist the Value Objects as Entities. In terms of the Domain, those objects would still be Value Objects but we will need to give them an id and relate them in a “one-to-many”/”one-to-one” relation with the owner, a real Entity. To summarise, your ORM handles the collection of Value Objects as Entities, but in your Domain they are still treated as Value Objects.

The main idea behind the “Join Table” strategy is to create a table that connects the owner Entity and its Value Objects. Let’s see a database representation.

如果需要持久化具有值对象集合的实体并需要查询功能，则可以选择将值对象持久化为实体。就域而言，这些对象仍然是值对象，但是我们需要给它们一个id，并以“一对多”/“一对一”的关系将它们与所有者\(一个真实的实体\)关联起来。总之，ORM将值对象集合作为实体处理，但是在您的域中它们仍然被视为值对象。



“联接表”策略背后的主要思想是创建一个连接所有者实体及其值对象的表。让我们看看数据库表示。

```php
CREATE TABLE `historical_products` (
`id`  varchar(255)  COLLATE  utf8_unicode_ci  NOT  NULL,
`name`  varchar(255)  COLLATE  utf8_unicode_ci  NOT  NULL,
`price_amount`  int(11)  NOT  NULL,
`price_currency`  varchar(255)  COLLATE  utf8_unicode_ci  NOT  NULL, PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

historical\_products table will look the same as products. Remember that HistoricalProduct extends Product Entity in order to easily show how to deal with persisting an collection. A new prices table is now required in order to persist all the different Money Value Objects that a Product Entity can handle.

historical\_products表看起来与products相同。请记住，HistoricalProduct扩展产品实体是为了方便地展示如何处理持久化集合。现在需要一个新的价格表来保存产品实体可以处理的所有不同的货币值对象。

```php
CREATE TABLE `prices` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`amount`  int(11)  NOT  NULL,
`currency`  varchar(255)  COLLATE  utf8_unicode_ci  NOT  NULL, PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

Finally, a table that relates products and prices is needed.

最后，需要一个关联产品和价格的表。

```php
CREATE TABLE `products_prices` (
`product_id`  varchar(255)  COLLATE  utf8_unicode_ci  NOT  NULL,
`price_id`  int(11)  NOT  NULL,
PRIMARY KEY (`product_id`,`price_id`),
UNIQUE KEY `UNIQ_62F8E673D614C7E7` (`price_id`),
KEY `IDX_62F8E6734584665A` (`product_id`),
CONSTRAINT `FK_62F8E6734584665A` FOREIGN KEY (`product_id`) REFERENCES `histor\ ical_products` (`id`),
CONSTRAINT `FK_62F8E673D614C7E7` FOREIGN KEY (`price_id`) REFERENCES `prices` \ (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```



