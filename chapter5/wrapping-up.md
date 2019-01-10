As we’ve seen, Services represent operations inside our system. We can differentiate between:



* Application Services: Help coordinate requests from the outside world into the domain. These Services should not contain domain logic. Transactions are handled in the application level, wrapping your services inside Transactional decorators will make your code transaction- agnostic.
* Domain Services: Operate with domain concepts only, those expressed by the Ubiquitous Language. Remember to postpone implementation details and think in behaviour first, Domain Services abuse will lead to anaemic domain models and bad Object-Oriented Design.

* Infrastructure Services: Operate over infrastructure like sending emails or logging informa- tion.



