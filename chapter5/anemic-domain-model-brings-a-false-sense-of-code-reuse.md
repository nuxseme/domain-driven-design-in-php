Say there is an instance were a client bypasses the UpdateOrderAmountService and instead fetches, updates and persists directly to the OrderRepository. If the UpdateOrderAmountService included any other relevant business logic regarding the order amount, it would not have be executed. This could lead to the order being stored in an inconsistent state. As such, invariants should be correctly guarded, and the best way to do this is to let the true domain model handle it. In the case of this example the Order entity would be the best place to ensure this.

```php
class Order
{
// ...

    public function changeAmount($amount)
    {
        $this->amount  =  $amount;
        $this->setUpdatedAt(new DateTimeImmutable());
    }
}
```

Note that by pushing this action down into the entity and naming it in terms of the Ubiquitous Language, the system achieves great code reuse. Anyone who now wishes to change the amount of the order has to invoke the Order::changeAmount method directly.

This leads to far richer classes, were behaviour is the ideal direction to aim for resulting code reuse. This is commonly referred to as a rich domain model.

