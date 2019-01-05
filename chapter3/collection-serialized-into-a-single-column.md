Serializing a collection of Value Objects into a single column is most likely the easiest solution. Everything that has previously been discussed through persisting a single Value Object applies in this situation. With Doctrine you can use an Object or Custom Type, with some additional considerations to bear in mind: Value Objects should be small in size, however, if you wish to persist a large collection, be sure to consider the maximum column length and the max row width that your database engine can handle.



> Exercise
>
> Think up both Doctrine Object Type and Doctrine Custom Type implementation strategies for persisting a Product with different prices.



