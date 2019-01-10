Caution must be had to not overuse domain service abstractions within your system. Following this path can lead to entities and value objects stripped of all behaviour, becoming mere data containers. This is contrary to the goal of OOP, which can be thought of as the gathering of both data and behaviour into semantic units called objects. Their intent being to express real-world concepts and problems. This can be considered an anti-pattern and is referenced to as the Anemic domain model.

Typically when starting a new project or feature, it is easy to fall into the trap of modeling the data first. This commonly includes thinking that each database table has a direct one-to-one object form representation. This thinking may or may not however be the exact case all the time.

Suppose we are task with modeling an order processing system. If we do start by modeling the data first, we could end up with an SQL script like so

```php
CREATE TABLE `orders` (
`ID`  INTEGER  NOT  NULL  AUTO_INCREMENT,
`CUSTOMER_ID`  INTEGER  NOT  NULL,
`AMOUNT`  DECIMAL(17,  2)  NOT  NULL  DEFAULT  '0.00',
`STATUS`  TINYINT  NOT  NULL  DEFAULT  0,
`CREATED_AT`  DATETIME  NOT  NULL,
`UPDATED_AT`  DATETIME  NOT  NULL,
PRIMARY KEY (`ID`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATION;
```

From this, it is relatively easy to create an Order class representation. This representation includes the required accessor methods, used to set/get data from and to the underlying orders database table.

```php
class Order
{
    const  STATUS_CREATED  = 10; 
    const  STATUS_ACCEPTED =  20; 
    const  STATUS_PAID	=  30; 
    const  STATUS_PROCESSED = 40;

    private $id;
    private $customerId; 
    private  $amount; 
    private $status; 
    private $createdAt; 
    private $updatedAt;

    public function construct(
        $customerId,
        $amount,
        $status,
        DateTimeInterface $createdAt, DateTimeInterface $updatedAt
    ) {
        $this->customerId = $customerId;
        $this->amount  =  $amount;
        $this->status   = $status;
        $this->createdAt   =  $createdAt;
        $this->updatedAt   =  $updatedAt;
    }

    public function setId($id)
    {
        $this->id = $id;
    }

    public function getId()
    {
        return $this->id;
    }

    public function setCustomerId($customerId)
    {
        $this->customerId = $customerId;
    }

    public function getCustomerId()
    {
        return $this->customerId;
    }

    public function setAmount($amount)
    {
        $this->amount  =  $amount;
    }

    public function getAmount()
    {
        return  $this->amount;
    }

    public function setStatus($status)
    {
        $this->status = $status;
    }

    public function getStatus()
    {
        return $this->status;
    }

    public function setCreatedAt(DateTimeInterface $createdAt)
    {
        $this->createdAt = $createdAt;
    }

    public function getCreatedAt()
    {
        return $this->createdAt;
    }

    public function setUpdatedAt(DateTimeInterface $updatedAt)
    {
        $this->updatedAt = $updatedAt;
    }

    public function getUpdatedAt()
    {
        return $this->updatedAt;
    }
}
```

An example use-case for this implementation could be to update the order status, as follows

```php
// Fetch an order from the database
$anOrder = $orderRepository->find(1);

// Update order status
$anOrder->setStatus(Order::STATUS_ACCEPTED);

// Update updatedAt field
$anOrder->setUpdatedAt(new DateTimeImmutable());

// Save  the  order  to the database
$orderRepository->save($anOrder);
```

This code has a similar problem to the initial user authentication solution, in regard to code reuse. To resolve this issue, defenders of such practice suggest the use of a Service Layer³, making the operations explicit and reusable. This above implementation could now instead be encapsulated into a separate class.

```php
class ChangeOrderStatusService
{
    private $orderRepository;

    public function construct(OrderRepository $orderRepository)
    {
        $this->orderRepository = $orderRepository;
    }

    public function execute($anOrderId, $anOrderStatus)
    {
// Fetch an order from the database
        $anOrder = $this->orderRepository->find($anOrderId);

// Update order status
        $anOrder->setStatus($anOrderStatus);

// Update updatedAt field
        $anOrder->setUpdatedAt(new DateTimeImmutable());

// Save the order to the database
        $this->orderRepository->save($anOrder);
    }
}
```

Or in the case of updating an order amount

```php
class UpdateOrderAmountService
{
    private $orderRepository;

    public function construct(OrderRepository $orderRepository)
    {
        $this->orderRepository = $orderRepository;
    }

    public function execute($orderId, $amount)
    {
        $anOrder = $this->orderRepository->find(1);

        $anOrder->setAmount($amount);
        $anOrder->setUpdatedAt(new DateTimeImmutable());
        $this->orderRepository->save($anOrder);
    }
}
```

The client code would be drastically decreased into following clearly intentioned operation.

```
$updateOrderAmountService = new UpdateOrderAmountService($orderRepository);

$updateOrderAmountService->execute(1, 20.5);
```

Implementing this approach can result in a large degree of code re-usability. Someone who wishes to update the order amount simply only has to retrieve an instance of the UpdateOrderAmountService and invoke the execute method with the appropriate parameters.

However, choosing this path breaks the discussed object-oriented design principles, and incurs the costs of building a domain model without taking advantage of any of the benefits.





