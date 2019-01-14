An interesting way of executing Application Services is through a Command Bus library. A good one is Tactician¹⁰



> What is a Command Bus? The term is mostly used when we combine the Command pattern¹¹ with a service layer¹². Its job is to take a Command object \(which describes what the user wants to do\) and match it to a Handler \(which executes it\). This can help structure your code neatly.



Fair enough, our Application Services are the Service Layer and our Request objects looks pretty much like Commands.

Wouldn’t be great if we had a mechanism to link all the Application Services and then based on the Request execute the correct one? Well, that is actually what a Comment Bus is.



