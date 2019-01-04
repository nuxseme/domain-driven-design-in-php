Consider the following code snippet

```php
$a  =  10;
$b  =  10; var_dump($a == $b);
// bool(true)
var_dump($a === $b);
// bool(true)
$a  =  20; var_dump($a);
// integer(20)
$a  =  $a  +  30; var_dump($a);
// integer(50)
```

Although $a and $b are different variables, stored low-level in different memory locations, when compared they are the same. They hold the same value. We consider them equal. You can change the value of $a from 10 to 20 at anytime you want, the new value is 20 and 10 has disappeared. You can replace integer values as much as you want without consideration of the previous value because you are not modifying it, you are just replacing it. If you apply any operation on them such as addition,

$a + $b, you get another new value that can be assigned to another variable or a previously defined one. When you pass $a to another function, except if explicitly passed by reference, you are passing a value. It does not matter if $a gets modified within that function because in your current code, you will still have the original copy. Value Objects behave as basic types.



虽然$a和$b是不同的变量，存储在不同的内存位置，但是它们在比较时是相同的。它们的价值相同。我们认为他们是平等的。您可以随时将$a的值从10更改为20，新的值是20，而10已经消失了。您可以任意替换整数值，而不需要考虑前面的值，因为您没有修改它，您只是替换它。如果你对它们应用任何运算，比如加法，

$a + $b，你会得到另一个新值，这个新值可以赋值给另一个变量或者之前定义的变量。当您将$a传递给另一个函数时，除非通过引用显式传递，否则您传递的是一个值。如果在该函数中修改了$a，这并不重要，因为在当前代码中，您仍然拥有原始副本。值对象表现为基本类型。

