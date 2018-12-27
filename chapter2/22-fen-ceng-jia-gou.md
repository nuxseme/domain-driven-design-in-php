From the code maintainability and reuse perspectives, the best way to make this code a bit easier to

maintain would be splitting up concepts - creating layers for each different concern. In our previous

example, it is easy to shape some different layers like the one encapsulating the data access and

manipulation, another one to handle infrastructure concerns and a final one for encapsulating the

orchestration of the previous two. An essential rule of the Layered architecture is that each layer

may be tightly coupled with the layers beneath it, as shown in the following picture

![](/assets/layered-architecture.png)

What the layered architecture really seeks is the separation of the different components of an

application. For instance, in terms of the previous example, a blog post representation must be

completely independent of a blog post as a conceptual entity. A blog post as a conceptual entity

can instead be associated with one or more representations, as opposed to being tightly coupled to

a specific representation. This is commonly named Separation of Concerns.

Another architecture paradigm and pattern that seeks the same purpose is the Model-View-Controller

pattern. It was initially thought and widely-used for building desktop GUI applications, and now it is

mainly used in web applications thanks to popular web frameworks like Symfony, Zend Framework

or Codeigniter.

