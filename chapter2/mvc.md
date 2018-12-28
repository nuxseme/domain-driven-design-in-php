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

> MVC是一种架构模式和范例，它将应用程序划分为三个主要的层:
>
> * model: 处理模型的行为。该层管理所有的数据，逻辑和业务。是MVC的核心
>
> * controller:协调其他层之间的交互。触发模型上的操作，以更新其状态并刷新与模型关联的表示。此外，控制器还可以向视图层发送消息，以更改特定的模型表示
>
> * view:模型层的不同表示，并提供一种方法来触发对模型状态的更改。

![](/assets/mvc.png)

---

* model层任务太重 
* 基础设施层未设定
* 控制层主要负责编排，数据流转



