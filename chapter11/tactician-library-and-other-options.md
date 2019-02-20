Tactician is a Command Bus library. It allows you to use the Command Pattern for your Application Services. It’s specially convenient for Application Services but you could use any kind of input tho.

Let’s see an example from the Tactician website

战术是一个命令总线库。它允许您为应用程序服务使用命令模式。它对于应用程序服务特别方便，但是您可以使用任何类型的输入tho。

让我们看一个来自战术家网站的例子

```php
// You build a simple message object like this:
class PurchaseProductCommand
{
    protected $productId;
    protected $userId;
// ...and constructor to assign those properties...
}
// And a Handler class that expects it:
class PurchaseProductHandler
{
    public function handle(PurchaseProductCommand $command)
    {
// use command to update your models, etc
    }
}
// And then in your controllers, you can fill in the command using your favorite
// form or serializer library, then drop it in the CommandBus and you're done!
$command = new PurchaseProductCommand(42, 29);
$commandBus->handle($command);
```

That’s it. Tactician is the $commandBus service. It does all the plumbing for finding the right handler and method. This can avoid a lot of boilerplate code. Here Commands and Handlers are just normal classes but you can configure whatever fits better your app.

Summarising, we can conclude that Commands are just Request objects and Command Handlers are just Application Services.

A cool thing about Tactician \(and Command Buses in general\) is that they are really easy to extend. Tactician provides plugins for common tasks like logging and database transactions. That way you can forget about setting up the wiring on every handler.

Another interesting plugin for Tactician is Bernard¹³ integration. Bernard is an asynchronous job queue that allows you to leave some tasks for later processing. Heavy processes block the user response and most of the time could be delayed for later processing. Answer the user fast and let the know once the job is done.

Mathias Noback developed a cool alternative to Tactician called SimpleBus check it out¹⁴.



就是这样。谋士是$commandBus服务。它为找到正确的处理程序和方法做了所有的准备工作。这可以避免许多样板代码。这里的命令和处理程序只是普通的类，但是您可以配置任何更适合您的应用程序的类。

总之，我们可以得出这样的结论:命令只是请求对象，命令处理程序只是应用程序服务。

关于战术专家\(以及一般的命令总线\)的一个很酷的事情是，它们真的很容易扩展。Tactician为日志和数据库事务等常见任务提供插件。这样您就可以忘记在每个处理程序上设置连接。

战术家的另一个有趣插件是Bernard integration。Bernard是一个异步作业队列，允许您将一些任务留到以后处理。繁重的进程会阻塞用户响应，并且大部分时间可能会被延迟以供以后处理。快速回答用户，并在工作完成后通知用户。

Mathias Noback开发了一个很酷的替代战术，叫做SimpleBus。

