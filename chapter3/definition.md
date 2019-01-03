Ward Cunningham defines¹ a Value Object as a measure or description of something. Examples of value objects are things like numbers, dates, monies and strings. Usually, they are small objects which are used quite widely. Their identity is based on their state rather than on their object identity. This way, you can have multiple copies of the same conceptual value object. Every $5 note has its own identity \(thanks to its serial number\), but the cash economy relies on every $5 note having the same value as every other $5 note.

Martin Fowler defines² a Value Object as a small object such as a Money or date range object. Their key property is that they follow value semantics rather than reference semantics. You can usually tell them because their notion of equality isn’t based on identity, instead two value objects are equal if all their fields are equal. Although all fields are equal, you don’t need to compare all fields if a subset is unique - for example currency codes for currency objects are enough to test equality. A general heuristic is that value objects should be entirely immutable. If you want to change a value object you should replace the object with a new one and not be allowed to update the values of the value object itself - updatable value objects lead to aliasing problems.





