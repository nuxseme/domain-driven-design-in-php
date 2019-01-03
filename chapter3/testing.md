Value Objects are tested in the same way normal objects are. However, the immutability and side- effect-free behaviour must be tested too. A solution is to create a copy of the Value Object you are testing before performing any modifications. Assert both are equal using the implemented equality check. Perform the actions you want to test and assert the results. Finally, assert that the original object and copy are still equal. Let’s put this into practice and test the side-effect-free implementation of our add method in the Money class.

```php

class MoneyTest extends \PHPUnit_Test_TestCase
{
    /**
     * @test
     */
    public function copiedMoneyShouldRepresentSameValue()
    {
        $aMoney  =  new  Money(100,  new  Currency('USD'));

        $copiedMoney  =  Money::fromMoney($aMoney);

        $this->assertTrue($aMoney->equals($copiedMoney));
    }

    /**
     * @test
     */
    public function originalMoneyShouldNotBeModifiedOnAddition()
    {
        $aMoney  =  new  Money(100,  new  Currency('USD'));

        $aMoney->add(new  Money(20,  new  Currency('USD')));

        $this->assertEquals(100,  $aMoney->amount());
    }

    /**
     * @test
     */
    public function moneysShouldBeAdded()
    {
        $aMoney  =  new  Money(100,  new  Currency('USD'));
        
        $newMoney  =  $aMoney->add(new  Money(20,  new  Currency('USD')));

        $this->assertEquals(120,  $newMoney->amount());
    }

// ...
}
```



