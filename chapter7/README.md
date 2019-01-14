When you place some classes together in a Module, you are telling the next developer who looks at your design to think about them together. If your model is telling a story, the Modules are chapters.

Eric Evans, Domain-Driven Design



A common concern when building an application following DDD, is where do we put the code? What’s the recommended way to place the code into the application? Where do we place infrastructure code? And more important, how should the different concepts inside the model be structured?

There’s a tactical pattern for this: modules. Nowadays, everyone structures the code in modules. But DDD goes ones step further and no technical concerns are considered when using modules. Indeed, it treats modules as a part of the model.



> Modules should not be treated as a way to separate code but as a way to separate meaningful concepts in the model.



