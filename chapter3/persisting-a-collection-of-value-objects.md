Imagine that now we would like to add to our Product Entity a collection of prices to be persisted. These prices could represent the different prices the product has bore throughout its lifetime, or the product price in different currencies. This could be named HistoricalPrice as shown below.



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

As in the previous sections, Serialization is a plausible approach if you do not care about querying capabilities, however, Embedded Values should be a possibility if we know exactly how many prices we want to persist. But, what happens if we want to persist a undetermined collection of historical prices?

