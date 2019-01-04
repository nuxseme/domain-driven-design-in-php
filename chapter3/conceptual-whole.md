So you may be thinking, why not just implement something similar to the following example, avoiding the need for a new Value Object class altogether?



因此，您可能会想，为什么不直接实现与下面的示例类似的东西，从而完全避免对新值对象类的需要

```php
class Product
{
    private $id;
    private  $name;

    /**
     * @var int
     */
    private  $amount;

/**
 * @var string
 */
    private $currency;

// ...
}
```

This approach has some noticeable flaws, if say for example you want to validate the ISO. It does not really make sense for the Product to be responsible for the currency’s ISO validation \(breaking the Single Responsibility Principle\). This is highlighted even more so if you want to reuse the accompanying logic in other parts of your domain \(to abide by the DRY principle\). With these factors in mind, this use-case is a perfect candidate to be abstracted out into a Value Object. Using this abstraction not only gives you the opportunity to group related properties together, but also to create higher-order concepts and a more concrete Ubiquitous Language.



这种方法有一些明显的缺陷，例如您想验证ISO。产品负责货币的ISO验证\(违反了单一责任原则\)实际上没有意义。如果您希望在域的其他部分重用附带的逻辑\(以遵守DRY原则\)，那么这一点将更加突出。考虑到这些因素，这个用例是一个完美的候选对象，可以抽象为一个值对象。使用这种抽象不仅可以将相关属性分组在一起，还可以创建更高阶的概念和更具体的通用语言



> Exercise
>
> Discuss with a peer if an email could be considered a Value Object or not. Does the context it is used in matter?
>
> 与同行讨论email是否可以被视为一个值对象。它所使用的上下文重要吗?



