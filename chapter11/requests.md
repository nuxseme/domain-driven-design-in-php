We need to send the email and password to the Application Service. There are many ways of doing such a thing from the client \(HTML form, API client or even the command-line\). We could just send standard parameters \(email and password\) through the method signature or build and send a data structure with this information. The later approach, sending a DTO¹, bring some interesting features to the table. By sending an object, it will be possible to serialise and queue it over a command bus. It will be possible to add type safety and some IDE help too.

我们需要将邮件和密码发送到应用程序服务。从客户机\(HTML表单、API客户机甚至命令行\)执行此类操作的方法有很多。我们可以通过方法签名发送标准参数\(电子邮件和密码\)，也可以使用这些信息构建和发送数据结构。后一种方法是发送DTO，它提供了一些有趣的特性。通过发送对象，可以通过命令总线对其进行序列化和排队。还可以添加类型安全性和一些IDE帮助。

> Data Transfer Object
>
> A DTO is a data structure that carries information between processes. Don’t mistake it with
>
> a full-featured object. A DTO does not have any behavior except for storage and retrieval of its own data \\(accessors and mutators\\). DTOs are simple objects that should not contain any business logic that would require testing.
>
> DTO是在进程之间传输信息的数据结构。别搞错了一个全功能的对象。除了存储和检索自己的数据\(访问器和变量\)之外，DTO没有任何行为。dto是简单的对象，不应该包含任何需要测试的业务逻辑。

```
As Vaughn Vernon says
```

> Application Service method signatures use only primitive types \(int, strings, etc.\), and possibly DTOs. As an alternative to these approaches, however, a better approach may be to design Command objects instead. There is not necessarily a right or wrong way. It mostly depends on your tastes and goals.
>
> 应用程序服务方法签名仅使用基本类型\(int、字符串等\)，也可能使用dto。然而，作为这些方法的替代方法，更好的方法可能是设计命令对象。不一定有正确或错误的方法。这主要取决于你的品味和目标。

The implementation for a DTO that holds the data required for the Application Service could be something like

保存应用程序服务所需数据的DTO的实现可以是这样的

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

如您所见，SignUpUserRequest根本没有行为，只有数据。这可能来自HTML表单或API端点，我们不在乎。

