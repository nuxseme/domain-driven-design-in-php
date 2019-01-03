So you may be thinking, why not just implement something similar to the following example, avoiding the need for a new Value Object class altogether?

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



> Exercise
>
> Discuss with a peer if an email could be considered a Value Object or not. Does the context it is used in matter?



