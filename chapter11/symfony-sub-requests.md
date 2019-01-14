If you use Symfony, Sub Requests⁸ could be an interesting option. Extracted from the Symfony site:



> In addition to the main request that’s sent into HttpKernel::handle, you can also send so-called sub request. A sub request looks and acts like any other request, but typically serves to render just one small portion of a page instead of a full page. You’ll most commonly make sub-requests from your controller \(or perhaps from inside a template, that’s being rendered by your controller\). This creates another full request-response cycle where this new Request is transformed into a Response. The only difference internally is that some listeners \(e.g. security\) may only act upon the master request. Each listener is passed some sub-class of KernelEvent, whose isMasterRequest\(\) can be used to check if the current request is a master or sub request.



This is great as you’ll get the benefits of invoking separate Application Services without AJAX penalty nor complicated ESI configurations.



