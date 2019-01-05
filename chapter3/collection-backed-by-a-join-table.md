In the case of needing to persist an Entity with a collection of Value Objects and need querying capabilities, you have the choice to persist the Value Objects as Entities. In terms of the Domain, those objects would still be Value Objects but we will need to give them an id and relate them in a “one-to-many”/”one-to-one” relation with the owner, a real Entity. To summarise, your ORM handles the collection of Value Objects as Entities, but in your Domain they are still treated as Value Objects.

The main idea behind the “Join Table” strategy is to create a table that connects the owner Entity and its Value Objects. Let’s see a database representation.

```php
CREATE TABLE `historical_products` (
`id`  varchar(255)  COLLATE  utf8_unicode_ci  NOT  NULL,
`name`  varchar(255)  COLLATE  utf8_unicode_ci  NOT  NULL,
`price_amount`  int(11)  NOT  NULL,
`price_currency`  varchar(255)  COLLATE  utf8_unicode_ci  NOT  NULL, PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

historical\_products table will look the same as products. Remember that HistoricalProduct extends Product Entity in order to easily show how to deal with persisting an collection. A new prices table is now required in order to persist all the different Money Value Objects that a Product Entity can handle.

```php
CREATE TABLE `prices` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`amount`  int(11)  NOT  NULL,
`currency`  varchar(255)  COLLATE  utf8_unicode_ci  NOT  NULL, PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

```

Finally, a table that relates products and prices is needed.



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



