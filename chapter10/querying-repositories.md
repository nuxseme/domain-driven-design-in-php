Upon comparison, repositories are different than a Collection if we consider their querying ability. A Repository deals with a large set of objects that typically are not in memory when the query is performed. It is not feasible to load all the instances of a domain object in memory and perform a query over them.

A good solution is to pass a criterion and let the Repository handle the implementation details to successfully perform the operation. It might translate the criterion to SQL, ORM queries or iterate over an in-memory collection, it does not matter, the implementation deals with it.

相比之下，如果考虑到存储库的查询能力，则存储库与集合是不同的。存储库处理大量对象，这些对象在执行查询时通常不在内存中。在内存中加载域对象的所有实例并对其执行查询是不可行的。

一个好的解决方案是传递一个标准，并让存储库处理实现细节，从而成功地执行操作。它可以将标准转换为SQL、ORM查询或在内存集合上进行迭代，这无关紧要，实现会处理它。

