Doctrine³ is a set of libraries for database storage and object mapping. It comes bundled with the popular Symfony 2 web framework⁴ by default and, among other features, it allows you to decouple your application from the persistence layer easily thanks to the Data Mapper pattern⁵.

The Object Relational Mapper stands over a powerful database abstraction layer that enables database interaction through a SQL dialect called Doctrine Query Language, inspired by the famous Java Hibernate framework.

If we are going to use Doctrine ORM the first task to complete is adding the dependencies to our project through Composer⁶

Doctrine的库是一组数据库存储和对象映射。它是与流行的Symfony 2绑定的web框架⁴默认情况下,在其他特性,它允许您将应用程序从数据持久层很容易由于⁵Mapper模式。

对象关系映射器位于一个强大的数据库抽象层之上，该抽象层支持通过一种称为Doctrine查询语言的SQL方言进行数据库交互，该SQL方言的灵感来自著名的Java Hibernate框架。

如果我们要使用原则ORM来完成第一个任务是通过Composer依赖项添加到我们的项目

composer require doctrine/orm:~2.4

