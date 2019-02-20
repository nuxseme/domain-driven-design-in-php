Along with other important features in PHP 5.3, namespaces entered the scene. This was a major shift, a group of the most important framework collaborators emerged with PHP-FIG², an acronym of PHP Framework Interop Group in an attempt to standardize and unify common aspects of the framework and library creation. The first PHP Standard Recommendation \(PSR, from now on\) that the group released was an autoloading standard that, summing up, proposes a one to one relation between a class and a PHP file using namespaces. Nowadays PSR-4, a simplification of PSR-0 that still maintains the relation between classes and physical PHP files, is the preferred and recommended way to structure code, and we believe that this should be the one used to implement Modules in a project. Returning to the previous example

除了PHP 5.3中的其他重要特性外，还引入了名称空间。这是一个重大转变,最重要的一组框架合作者出现PHP-FIG², PHP开发框架交互操作组的首字母缩写,试图规范和统一公共方面的框架和库的创建。该小组发布的第一个PHP标准建议\(PSR，从现在开始\)是一个自动加载标准，总结起来就是使用名称空间在类和PHP文件之间提出一对一的关系。现在，PSR-4是对PSR-0的简化，它仍然保持类和物理PHP文件之间的关系，是构建代码的首选和推荐方法，我们认为这应该是用于在项目中实现模块的方法。回到前面的例子

```
├── composer.json
├── composer.lock
└── src
└── BuyIt
└── Billing
└── DomainModel
└── Bill
└── Bill.php
```

The class name for the Bill entity, using namespaces and PSR-0/PSR-4, would become simply Bill and the full qualified class name would be BuyIt\Billing\DomainModel\Bill\Bill. As you can see, this way enables us to name domain objects in terms of the Ubiquitous Language and is the preferred way to structure and organize code.

使用名称空间和PSR-0/PSR-4的Bill实体的类名将成为简单的Bill，而完全限定的类名将是BuyIt\Billing\DomainModel\Bill\Bill。如您所见，这种方式使我们能够根据普遍存在的语言来命名域对象，并且是构造和组织代码的首选方法。

