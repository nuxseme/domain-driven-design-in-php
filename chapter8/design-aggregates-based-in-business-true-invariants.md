First of all, what’s an invariant? An invariant is a rule that must be true and be consistent during code execution. Let’s see an example to help you. A stack³ is a LIFO data structure where we can push elements, pop them and ask for its size. Consider a pure PHP implementation without using any specific PHP array functions such as array\_pop.



```php
class Stack
{
    private $data;
    public function __construct()
    {
        $this->data = [];
    }
    public function push($value)
    {
        $this->data[] = $value;
    }
    public function size()
    {
        return count($this->data);
    }
    /**
     * @return mixed
     */
    public function pop()
    {
        $topIndex = $this->size() - 1;
        $top = $this->data[$topIndex];
        unset($this->data[$topIndex]);
        return $top;
    }
}
```



Imagine that getting the size of the stack would be a really CPU intensive and high-cost call. There is an option to improve that method, introducing a private attribute to keep track of the number of elements in the internal array.



```


class Stack
{
    private $data;
    private $size;
    public function __construct()
    {
        $this->data = [];
        $this->size = 0;
    }
    public function push($value)
    {
        $this->data[] = $value;
        $this->size++;
    }
    public function size()
    {
        return $this->size;
    }
    /**
     * @return mixed
     */
    public function pop()
    {
        $topIndex = $this->size() - 1;
        $top = $this->data[$topIndex];
        unset($this->data[$topIndex]);
        $this->size--;
        return $top;
    }
}
```

With this approach, now asking for the size of the stack is a light method. What’s the cost? There were some updates in pop and push methods to keep track of the new size when adding and removing elements. Let’s try to find an invariant, a rule that should be true before and after any method call of the stack object. What about $this-&gt;size === count\($this-&gt;data\)? Before and after each call to any method in the stack object, the attribute size is consistent and it always holds the size of the stack.

So what’s a? An invariant is a business rule that must always be true and transactionally consistent. In the previous example, the amount of an Order must match the sum of amounts of all the Line Orders is an invariant. That makes probably the Order be the root and allow us to make operations to it rather than the Order Lines that are inside. With this approach, we have a single entry point to perform operations on the cluster that will be consistent and do not break any invariant. It means, there is no chance to invoke a method to break such rule. Each time you add a Line Order or update info from the root, internally, the Order amount gets calculated.

Be careful about the “has-a”/”has-many” relations that do not necessary make two Entities become one Aggregate with one of those being the root.





