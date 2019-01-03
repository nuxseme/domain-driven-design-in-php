Consider a Product Entity that contains a Money Value Object used to quantify its price. Consider also two Product Entities whose price is identical, for example 100 USD. This scenario could be modelled using two individual Money objects or two references pointing to a single Value Object.

Sharing the same Value Object can be risky, if one is altered, both will reflect the change. This behaviour can be considered an unexpected side-effect. For example, if Carlos was hired on February, 20th, and we know that Christian was also hired on the same day, we may set Christian’s hire date to be the same instance as Carlos’s. If Carlos then changes the month in his hire date to May, Christian’s hire date changes too. Whether it is correct or not, it is not what people expect.

Due to the problems highlighted in this example when holding a reference to a Value Object, rather than modifying its value, it is recommended instead to replace the object as a whole.

```php
$this->price  =  new  Money(100,  new  Currency('USD'));
// ...
$this->price = $this->price->increaseAmountBy(200);
```

This kind of behaviour is similar to how basic types such as strings work in PHP. Consider the function strtolower, it returns a new string rather than modifying the original one. No reference is used, but instead a new value is returned.



