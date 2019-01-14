We need to send the email and password to the Application Service. There are many ways of doing such a thing from the client \(HTML form, API client or even the command-line\). We could just send standard parameters \(email and password\) through the method signature or build and send a data structure with this information. The later approach, sending a DTO¹, bring some interesting features to the table. By sending an object, it will be possible to serialise and queue it over a command bus. It will be possible to add type safety and some IDE help too.

> Data Transfer Object
>
>     A DTO is a data structure that carries information between processes. Don’t mistake it with
>
>     a full-featured object. A DTO does not have any behavior except for storage and retrieval of its own data \(accessors and mutators\). DTOs are simple objects that should not contain any business logic that would require testing.

    As Vaughn Vernon says



> Application Service method signatures use only primitive types \(int, strings, etc.\), and possibly DTOs. As an alternative to these approaches, however, a better approach may be to design Command objects instead. There is not necessarily a right or wrong way. It mostly depends on your tastes and goals.



The implementation for a DTO that holds the data required for the Application Service could be something like

```php
namespace Lw\Application\Service\User;
class SignUpUserRequest
{
    private $email;
    private $password;
    public function __construct($email, $password)
    {
        $this->email = $email;
        $this->password = $password;
    }
    public function email()
    {
        return $this->email;
    }
    public function password()
    {
        return $this->password;
    }
}
```

As you see, SignUpUserRequest does not have behaviour at all, only data. This could have come from an HTML form or an API end-point, we don’t care.



