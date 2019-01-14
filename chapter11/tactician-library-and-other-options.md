Tactician is a Command Bus library. It allows you to use the Command Pattern for your Application Services. It’s specially convenient for Application Services but you could use any kind of input tho.

Let’s see an example from the Tactician website



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



