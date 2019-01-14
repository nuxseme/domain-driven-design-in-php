The best way to learn about Aggregates is seeing code. So, let’s consider the following scenario: A web application where users can make wishes that would like to be granted if something happened to them. Think about it as a will. For example, I would like to send an email to my wife explaining what to do with my GitHub account if I die in a horrible accident. They way to check that I’m still alive is to answer to emails the platform is sending to me. If you want to know more about this application⁴ you can visit our GitHub account⁵ with this example and more.

Ok, so we have users and their wishes. Let’s consider only one use case, “As a User, I want to make a Wish”. How could we model these two concepts? As the good practices when designing Aggregates, let’s try to push for small Aggregates, that would be, in this case, two different Aggregates of one Entity each, User and Wish. What about the relation between them? Well, we should use an identifier, for example, UserId. Let’s see some code.



