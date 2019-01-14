Application requests are data structures not objects. Unit testing data structures is like testing getters and setters. There is no behaviour to test so there is not that much value trying to unit testing them. This structures will be covered as a side-effect of more elaborated tests like Integration or Acceptance tests.

An alternative to request objects are Commands. We could design an Service with multiple Application methods. Each one of them with the parameters you’d put inside the Request. It’s okay for simple applications. No worries, we’ll come back to this topic later.



