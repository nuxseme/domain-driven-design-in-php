After signing up, we might be thinking about redirecting the user to a profile page. The most immediate reflection for doing that on the controller could be returning the User entity directly from the service.

注册之后，我们可能会考虑将用户重定向到配置文件页面。在控制器上执行此操作的最直接反映是直接从服务返回用户实体。

```php
class SignUpUserService
{
// ...
    public function execute(SignUpUserRequest $request)
    {
// ...
        $user = new User(
            $this->userRepository->nextIdentity(),
            $email,
            $password
        );
        $this->userRepository->add($user);
        return $user;
    }
}
```

Then, from the controller, we would pick up the id field and redirect to some other place.

However, think twice about what we’ve just done. We’ve returned a full-featured entity to the controller. This will allow the delivery mechanism to bypass the Application Layer and interact directly with the the domain.

Imagine the User entity offers an updateEmailAddress method. You could try to prevent it, but at some point in the future, somebody might think about using it.

然后，从控制器中，我们将获取id字段并重定向到其他地方。

但是，仔细想想我们刚才所做的。我们已经向控制器返回了一个功能齐全的实体。这将允许交付机制绕过应用层，直接与域交互。

假设用户实体提供了一个updateEmailAddress方法。你可以试着阻止它，但是在将来的某个时候，有人可能会考虑使用它。

```php
$app->match('/signup', function (Request $request) use ($app) {
//...
    $user = $app['sign_up_user_application_service']->execute(
        new SignUpUserRequest(
            $request->get('email'),
            $request->get('password')
        )
    );
    $user->updateEmailAddress('shouldnotupdate@email.com');
//...
});
```

Not only that, the data that the presentation layer needs is not the same that the Domain manages. We don’t want to evolve and couple the domain layer around the presentation layer. We want to evolve them freely.

We need a way a flexible way of decoupling both layers.

不仅如此，表示层需要的数据与域管理的数据并不相同。我们不希望在表示层周围发展和耦合域层。我们想让它们自由发展。

我们需要一种灵活的方法来解耦这两层。

