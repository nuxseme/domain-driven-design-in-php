You should struggle to publish Domain Events from deeper in the chain. The nearer inside the Entity or the Value Object, the better. As we have seen in the previous section, sometimes this is not easy but the final result is simpler for the clients. We have seen developers publishing Domain Events from the Application Services or Domain Services. That seems an easier approach to implement but drives to an Anemic-Domain Model in the same way when pushing business logic to Domain Services rather than placing it into your Entities.



