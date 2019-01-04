Ward Cunningham defines¹ a Value Object as a measure or description of something. Examples of value objects are things like numbers, dates, monies and strings. Usually, they are small objects which are used quite widely. Their identity is based on their state rather than on their object identity. This way, you can have multiple copies of the same conceptual value object. Every $5 note has its own identity \(thanks to its serial number\), but the cash economy relies on every $5 note having the same value as every other $5 note.

Martin Fowler defines² a Value Object as a small object such as a Money or date range object. Their key property is that they follow value semantics rather than reference semantics. You can usually tell them because their notion of equality isn’t based on identity, instead two value objects are equal if all their fields are equal. Although all fields are equal, you don’t need to compare all fields if a subset is unique - for example currency codes for currency objects are enough to test equality. A general heuristic is that value objects should be entirely immutable. If you want to change a value object you should replace the object with a new one and not be allowed to update the values of the value object itself - updatable value objects lead to aliasing problems.

Examples of Value Objects are numbers, text strings, dates, times, a person’s full name \(composed of first, middle, last name, and title\), currencies, colours, phone numbers, and postal addresses.

Ward Cunningham¹值对象定义为衡量或描述。值对象的例子有数字、日期、货币和字符串。通常，它们都是小物件，使用非常广泛。它们的标识基于它们的状态，而不是它们的对象标识。这样，您可以拥有相同概念值对象的多个副本。每一张5美元纸币都有自己的身份\(这要归功于它的序列号\)，但现金经济依赖于每一张5美元纸币的价值都与其他5美元纸币相同。

Martin Fowler将²值对象定义为一个小物件,比如金钱或日期范围对象。它们的关键属性是遵循值语义而不是引用语义。通常可以告诉它们，因为它们的相等性概念不是基于恒等性的，而是两个值对象相等，如果它们的所有字段都相等。尽管所有字段都是相等的，但如果子集是唯一的，则不需要比较所有字段——例如，货币对象的货币代码足以测试相等。一般的启发式是，值对象应该是完全不可变的。如果您想要更改一个值对象，您应该用一个新的对象替换该对象，并且不允许更新值对象本身的值——可更新值对象会导致混叠问题。

值对象的例子有数字、文本字符串、日期、时间、人名\(由名、中、姓和标题组成\)、货币、颜色、电话号码和邮政地址。

> Exercise
>
> Try to locate more examples of potential Value Objects in your current Domain.

---

1. 依赖所描述的内容 不依赖对象的引用
2. ...



