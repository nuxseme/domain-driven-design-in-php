There are cases where might be a good idea to group of cohesive Application Services under the same class

在某些情况下，将内聚的应用程序服务分组在同一个类下可能是一个好主意

```php
class UserService
{
// ...

    public function signUp(SignUpUserRequest $request)
    {
// ...
    }

    public function signIn(SignUpUserRequest $request)
    {
// ...
    }

    public function logOut(LogOutUserRequest $request)
    {
// ...
    }
}
```

We don’t recommend such approach as not all Application Services are 100% cohesive. Some services will require different dependencies and you’ll end up with Application Services depending on things that they don’t need. Another issue is that this kind of class grows fast. As it violates the Single Responsibility Principle, there will be multiple reasons to change and maybe break it.

我们不推荐这种方法，因为不是所有的应用程序服务都是100%内聚的。有些服务将需要不同的依赖项，而您最终将得到应用程序服务，这取决于它们不需要的东西。另一个问题是这类类增长很快。因为它违反了单一责任原则，所以会有很多理由去改变甚至打破它。

