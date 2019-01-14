Creating a request from the delivery mechanism, your favourite framework, should be pretty straightforward. On web, you could pick up parameters from the controller request and pass them down to the service inside a DTO. Same principle applies for a CLI command, read input parameters and send them down again.

With Symfony, we can extract the data we need from Request object from the HttpFoundation component.

```php
//...
class UsersController extends Controller
{
    /**
     * @Route("/signup", name="signup")
     * @param Request $request
     * @return Response
     */
    public function signUpAction(Request $request)
    {
//...
        $SignUpUserRequest = new SignUpUserRequest(
            $request->get('email'),
            $request->get('password')
        );
//...
    }
//...
```

On a more elaborated Silex application that uses the Form component to capture and validate parameters



```php

$app->match('/signup', function (Request $request) use ($app) {
    $form = $app['sign_up_form'];
    $form->handleRequest($request);
    if ($form->isValid()) {
        $data = $form->getData();
        try {
            $app['sign_in_user_application_service']->execute(
                new SignUpUserRequest(
                    $data['email'],
                    $data['password']
                )
            );
            return $app->redirect($app['url_generator']->generate('login'));
        } catch (UserAlreadyExistsException $e) {
            $form
                ->get('email')
                ->addError(
                    new FormError(
                        'Email is already registered by another user'
                    )
                );
        } catch (\Exception $e) {
            $form
                ->addError(
                    new FormError(
                        'There was an error, please get in touch with us'
                    )
                );
        }
    }
    return $app['twig']->render('signup.html.twig', [
        'form' => $form->createView(),
    ]);
});
```



