If you use Symfony, Sub Requests⁸ could be an interesting option. Extracted from the Symfony site:

> In addition to the main request that’s sent into HttpKernel::handle, you can also send so-called sub request. A sub request looks and acts like any other request, but typically serves to render just one small portion of a page instead of a full page. You’ll most commonly make sub-requests from your controller \(or perhaps from inside a template, that’s being rendered by your controller\). This creates another full request-response cycle where this new Request is transformed into a Response. The only difference internally is that some listeners \(e.g. security\) may only act upon the master request. Each listener is passed some sub-class of KernelEvent, whose isMasterRequest\(\) can be used to check if the current request is a master or sub request.

This is great as you’ll get the benefits of invoking separate Application Services without AJAX penalty nor complicated ESI configurations.



如果使用Symfony，子请求可能是一个有趣的选项。从Symfony位点提取:

> 除了发送到HttpKernel::handle的主请求之外，您还可以发送所谓的子请求。子请求的外观和行为与任何其他请求类似，但通常只呈现页面的一小部分，而不是整个页面。您通常会从控制器发出子请求\(或者可能从控制器呈现的模板内部发出\)。这将创建另一个完整的请求-响应循环，其中将新请求转换为响应。内部惟一的不同是，一些侦听器\(例如安全性\)可能只根据主请求进行操作。每个侦听器都被传递一些KernelEvent的子类，其isMasterRequest\(\)可用于检查当前请求是主请求还是子请求。

这很好，因为您将获得调用独立应用程序服务的好处，而不会受到AJAX的影响，也不会遇到复杂的ESI配置。

