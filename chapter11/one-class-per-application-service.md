Our preferred approach, probably the one that fits all scenarios.



```php
class SignUpUserService
{
// ...
    public function execute(SignUpUserRequest $request)
    {
// ...
    }
}
```

Using a dedicated class per Application Service makes it more robust to external changes \(Single Responsibility Principle\). There are fewer reasons to change the class as the service does one and only one thing. The Application Service will be easier to test as it does less things. It’s easier to implement a common Application Service contract, making class decoration easier \(check out transactions in repositories chapter\). There is greater cohesion with the dependencies as all dependencies are dedicated the Application Service.

The execution method could have a more expressive name, like signUp. However, the execute Command Pattern⁵ format standardises a common contract across Application Services enabling easy decoration. Handy for transactions.





