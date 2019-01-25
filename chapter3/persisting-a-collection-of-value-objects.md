Imagine that now we would like to add to our Product Entity a collection of prices to be persisted. These prices could represent the different prices the product has bore throughout its lifetime, or the product price in different currencies. This could be named HistoricalPrice as shown below.

假设现在我们要向产品实体添加一个要持久化的价格集合。这些价格可以代表产品一生中承受的不同价格，或者以不同货币表示的产品价格。可以将其命名为HistoricalPrice，如下所示。

```php
class HistoricalProduct extends Product
{
    /**
     * @var Money[]
     */
    protected $prices;

    public  function  construct($aProductId,  $aName,  Money  $aPrice,  array  $someP\ rices)
    {
    parent::construct($aProductId,  $aName,  $aPrice);
    $this->setPrices($somePrices);
    }

    private function setPrices(array $somePrices)
    {
        $this->prices = $somePrices;
    }

    public function prices()
    {
        return $this->prices;
    }
}
```

HistoricalProduct extends from Product so it inherits the same behaviour plus the price collection functionality.

HistoricalProduct从产品扩展而来，因此它继承了相同的行为和价格收集功能。

As in the previous sections, Serialization is a plausible approach if you do not care about querying capabilities, however, Embedded Values should be a possibility if we know exactly how many prices we want to persist. But, what happens if we want to persist a undetermined collection of historical prices?



正如前几节中所述，如果您不关心查询功能，序列化是一种可行的方法，但是，如果我们确切地知道希望保留多少价格，则应该可以使用嵌入值。但是，如果我们想要维持一个不确定的历史价格集合，会发生什么呢?

