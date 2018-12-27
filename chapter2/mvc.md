Model-View-Controller is an architectural pattern and paradigm that divides the application into

three main layers:

• The Model: Captures and centralizes all the domain model behaviour. This layer manages all

the data, logic and business rules independently of the data representation. It can be said that

the Model layer is the heart and soul of every MVC application.

• The Controller: Orchestrates interactions between the other layers. Triggers actions on the

model in order to update its state and refreshes the representations associated to the model.

Additionally, the Controller can also send messages to the View layer in order to change the

specific Model representation.

• The View: A layer whose main purpose is to expose the differing representations of the Model

layer and to give a way to trigger changes on the Model’s state.

![](/assets/mvc.png)

