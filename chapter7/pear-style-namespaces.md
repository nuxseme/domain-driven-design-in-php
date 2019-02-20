Before PHP 5.3, due to the lack of a namespace construction, PEAR-style namespaces were used. PEAR is the acronym for PHP Extension and Application Repository and in the good old times was a repository of reusable components. It’s still active, but its use is a minority and there’s a lot of unmaintained packages. Especially since composer and packagist took the stage. PEAR, as a source of reusable components, needed a way to avoid class name collisions so they started prefixing class names with namespaces. There are still projects that use this form of namespaces \(PHPUnit or Zend Framework 1, to name a few\). The following would be an example of PEAR-style namespaces

在PHP 5.3之前，由于缺少名称空间构造，所以使用了pearstyle名称空间。PEAR是PHP扩展和应用程序存储库的首字母缩写，在过去是可重用组件的存储库。它仍然是活动的，但是它的使用是少数的，并且有很多未维护的包。特别是自从作曲家和包装工上台以来。PEAR作为可重用组件的来源，需要一种避免类名冲突的方法，因此它们开始使用名称空间作为类名的前缀。仍然有一些项目使用这种形式的名称空间\(例如PHPUnit或Zend Framework 1\)。下面是pearstyle名称空间的一个示例

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

使用pearstyle命名空间的Bill实体的类名将成为BuyIt\_- Billing\_DomainModel\_Bill\_Bill。这个类名有点难看，而且不遵循主要的DDD咒语之一:每个类名都应该使用通用语言来命名。因此，我们强烈反对使用它。

