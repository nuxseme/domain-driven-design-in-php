Consider a Product Entity that contains a Money Value Object used to quantify its price. Consider also two Product Entities whose price is identical, for example 100 USD. This scenario could be modelled using two individual Money objects or two references pointing to a single Value Object.

Sharing the same Value Object can be risky, if one is altered, both will reflect the change. This behaviour can be considered an unexpected side-effect. For example, if Carlos was hired on February, 20th, and we know that Christian was also hired on the same day, we may set Christian’s hire date to be the same instance as Carlos’s. If Carlos then changes the month in his hire date to May, Christian’s hire date changes too. Whether it is correct or not, it is not what people expect.

Due to the problems highlighted in this example when holding a reference to a Value Object, rather than modifying its value, it is recommended instead to replace the object as a whole.

考虑一个产品实体，它包含一个货币价值对象，用于量化其价格。再考虑两个价格相同的产品实体，例如100美元。可以使用两个单独的Money对象或指向单个值对象的两个引用来建模此场景。

共享相同的值对象是有风险的，如果其中一个对象被更改，那么两个对象都将反映更改。这种行为可以被认为是一种意外的副作用。例如，如果Carlos是在2月20日被雇佣的，并且我们知道Christian也是在同一天被雇佣的，那么我们可以将Christian的雇佣日期设置为与Carlos相同的实例。如果卡洛斯把他的雇佣日期改为五月，克里斯蒂安的雇佣日期也会改变。不管正确与否，这都不是人们所期望的。

由于本例中突出显示的问题，在保存对值对象的引用而不是修改其值时，建议将对象作为一个整体替换。

```php
$this->price  =  new  Money(100,  new  Currency('USD'));
// ...
$this->price = $this->price->increaseAmountBy(200);
```

This kind of behaviour is similar to how basic types such as strings work in PHP. Consider the function strtolower, it returns a new string rather than modifying the original one. No reference is used, but instead a new value is returned.

这种行为类似于PHP中字符串等基本类型的工作方式。考虑strtolower函数，它返回一个新字符串，而不是修改原来的字符串。不使用引用，而是返回一个新值。

