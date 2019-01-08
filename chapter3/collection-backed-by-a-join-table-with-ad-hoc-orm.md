It is possible to do the same with an ad-hoc ORM, where Cascade INSERTS and JOIN queries are required. The only consideration to be careful about is how removal of Value Objects are handled, in order not to leave orphan Money Value Objects.



> Exercise
>
> Think up a solution for DbalHistoricalRepository that would handle the persist method.



