There are cases where might be a good idea to group of cohesive Application Services under the same class



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



