“Database Entity” is the same strategy as “Join Table” with the addition that the Value Object is only managed by the owner Entity. In the current scenario, consider that the Money Value Object is only used by the HistoricalProduct Entity. A “Join Table” would be over-complex. So, the same result could be achieved using a “one-to-many” database relation.



> Exercise
>
> Think of the mapping required between HistoricalProduct and Money if a “Database Entity” approach was used.



