Factories are a powerful tool for decoupling construction logic from our business logic. The Factory Method pattern not only helps moving creation responsibility to the Aggregate Root but also could force domain invariants. Using the Abstract Factory pattern in our Services allows us to separate our domain logic from infrastructure creation details. A common use case for this are Specifications and their respective persistence implementations. We’ve seen that factories come very handy on our test suites too. While we could extract building logic into Object Mother Factories, Test Data Builders provide more flexibility for our tests.



