Avoid to put any business logic inside you request objects. Not even validation. Validation should happen inside your Domain – this is inside your Entities, Value Objects, Domain Services, etc. – as business invariants and constraints.

避免在请求对象中放入任何业务逻辑。没有验证。验证应该作为业务不变量和约束在域内进行——这是在实体、值对象、域服务等内部进行的。



