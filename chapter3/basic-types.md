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



