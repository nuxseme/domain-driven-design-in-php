Our preferred approach, probably the one that fits all scenarios.

我们首选的方法，可能是适合所有场景的方法。

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



为每个应用程序服务使用一个专用类使其对外部更改\(单一责任原则\)更加健壮。当服务只做一件事时，更改类的原因就更少了。应用程序服务将更容易测试，因为它做的事情更少。更容易实现公共应用程序服务契约，使类的修饰更容易\(查看repository一章中的transaction\)。由于所有依赖项都专用于应用程序服务，因此与依赖项的内聚性更强。



执行方法可以有一个更有表现力的名称，比如注册。然而，execute命令模式格式标准化了跨应用程序服务的公共契约，从而实现了简单的装饰。方便交易。

