Before PHP 5.3, due to the lack of a namespace construction, PEAR-style namespaces were used. PEAR is the acronym for PHP Extension and Application Repository and in the good old times was a repository of reusable components. It’s still active, but its use is a minority and there’s a lot of unmaintained packages. Especially since composer and packagist took the stage. PEAR, as a source of reusable components, needed a way to avoid class name collisions so they started prefixing class names with namespaces. There are still projects that use this form of namespaces \(PHPUnit or Zend Framework 1, to name a few\). The following would be an example of PEAR-style namespaces

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



The class name for the Bill entity, using the PEAR-style namespaces, would become BuyIt\_- Billing\_DomainModel\_Bill\_Bill. That class name it’s a bit ugly and don’t follow one of the main DDD mantras: every class name should be named in terms of Ubiquitous Language. For this reason we strongly discourage its usage.



