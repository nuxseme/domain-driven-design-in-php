Another interesting detail about modeling your Domain concepts using Value Objects is about its security benefits. Consider an application within a selling flight tickets context. If you deal with International Air Transport Association airport codes, also known as the IATA codes²⁰, you can decide to use a string or model yhe concept using a Value Object. If you choose to go with the string, think about all the places where you will be checking that the string is a valid IATA code. What’s the chance to forget anywhere important? On the other side, think about trying to instantiate a new IATA\("BCN'; DROP TABLE users;--"\). If you centralize the guards²¹ in the constructor and the pass into your model a IATA Value Object avoiding SQL Injections or similar attacks get easier.

If you want to know more about the security side of Domain-Driven Design, you can follow Dan Bergh Johnsson²² or read his blog²³.



