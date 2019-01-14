After signing up, we might be thinking about redirecting the user to a profile page. The most immediate reflection for doing that on the controller could be returning the User entity directly from the service.



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



