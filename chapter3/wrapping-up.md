Using Value Objects for modeling concepts in your Domain that measure, quantify or describe is highly recommended. As shown, Value Objects are easy to create, maintain and test. In order to handle persistence within a DDD application, using an ORM is a must. However, in order to persist Value Objects using Doctrine without current support for embedded values \(scheduled for version 2.5\), there are two options: adding the Value Object fields directly into your Entity and mapping them \(less elegant, but easier\) or extending your entities \(far more elegant, but more complex\).



