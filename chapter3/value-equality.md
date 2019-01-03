As discussed at the beginning of the chapter, two Value Objects are equal if the content they measure, quantify, or describe is the same.

For example, conceptualise two Money objects representing 1 USD. Can we consider them equal? In the ‘real’ world are two coins of 1 USD valued the same? Of course they are. Directing our attention back to the code, the Value Objects in question refers to separate instances of Money. However, we can consider them to both represent the same value, so in-turn they are equal.

In regard to PHP, it is common place to compare two Value Objects using the == operator. Examining the PHP documentations⁹ definition of the operator highlights an interesting behaviour.



When using the comparison operator \(==\), object variables are compared in a simple manner, namely: Two object instances are equal if they have the same attributes and values, and are instances of the same class.



This behaviour works in agreement to our formal definition of a Value Object. However, as an exact class match predicate is present, you should be wary when handling sub-typed Value Objects.

With this in mind, the even stricter === operator unfortunately does not help us.



When using the identity operator \(===\), object variables are identical if and only if they refer to the same instance of the same class.



The following example should help confirm these subtle differences.



