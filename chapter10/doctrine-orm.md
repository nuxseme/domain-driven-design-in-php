Doctrine³ is a set of libraries for database storage and object mapping. It comes bundled with the popular Symfony 2 web framework⁴ by default and, among other features, it allows you to decouple your application from the persistence layer easily thanks to the Data Mapper pattern⁵.

The Object Relational Mapper stands over a powerful database abstraction layer that enables database interaction through a SQL dialect called Doctrine Query Language, inspired by the famous Java Hibernate framework.

If we are going to use Doctrine ORM the first task to complete is adding the dependencies to our project through Composer⁶



composer require doctrine/orm:~2.4

