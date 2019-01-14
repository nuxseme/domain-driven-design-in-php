With the rise of Domain-Driven Design \(DDD\), architectures promoting domain centric designs are becoming more popular. This is the case with Hexagonal Architecture, also known as Ports and Adapters, that seems to have being rediscovered just now by PHP developers. Invented in 2005 by Alistair Cockburn, one of the Agile Manifesto authors, the Hexagonal Architecture allows an application to be equally driven by users, programs, automated tests or batch scripts, and to be developed and tested in isolation from its eventual run-time devices and databases. This results into agnostic infrastructure web applications that are easier to test, write and maintain. Let’s see how to apply it using real PHP examples.

Your company is building a brainstorming system called Idy. Users add and rate ideas so the most interesting ones can be implemented in a company. It’s Monday morning, another sprint is starting and you are reviewing some user stories with your team and your Product Owner. “As a not logged in user, I want to rate an idea and the author should be notified by email”, that’s a really important one, isn’t it?



