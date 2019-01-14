Along with other important features in PHP 5.3, namespaces entered the scene. This was a major shift, a group of the most important framework collaborators emerged with PHP-FIG², an acronym of PHP Framework Interop Group in an attempt to standardize and unify common aspects of the framework and library creation. The first PHP Standard Recommendation \(PSR, from now on\) that the group released was an autoloading standard that, summing up, proposes a one to one relation between a class and a PHP file using namespaces. Nowadays PSR-4, a simplification of PSR-0 that still maintains the relation between classes and physical PHP files, is the preferred and recommended way to structure code, and we believe that this should be the one used to implement Modules in a project. Returning to the previous example





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



