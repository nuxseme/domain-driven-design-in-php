Upon comparison, repositories are different than a Collection if we consider their querying ability. A Repository deals with a large set of objects that typically are not in memory when the query is performed. It is not feasible to load all the instances of a domain object in memory and perform a query over them.

A good solution is to pass a criterion and let the Repository handle the implementation details to successfully perform the operation. It might translate the criterion to SQL, ORM queries or iterate over an in-memory collection, it does not matter, the implementation deals with it.

