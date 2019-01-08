Some people understand validation as the process whereby a service validates the state of a given object. In this case, the validation conforms to a design-by-contract⁷ approach - consisting of pre- conditions, post-conditions and invariants. One such way to protect a single attribute is by using Value Objects. In order to make our design more flexible for change, we focus only on asserting domain pre-conditions that must be met. Here we will be using guards as an easy way of validating the pre-conditions:

```php
class Username
{
    const  MIN_LENGTH  =  5;
    const  MAX_LENGTH  =  10;
    const FORMAT = '/^[a-zA-Z0-9_]+$/';

    private  $username;

    public function construct($username)
    {
        $this->setUsername($username);
    }

    private setUsername($username)
    {
        $this->assertNotEmpty($username);
        $this->assertNotTooShort($username);
        $this->assertNotTooLong($username);
        $this->assertValidFormat($username);
        $this->username  =  $username;
    }

    private function assertNotEmpty($username)
    {
        if (empty($username)) {
            throw new InvalidArgumentException('Empty username');
        }
    }

    private function assertNotTooShort($username)
    {
        if  (strlen($username)  <  self::MIN_LENGTH)  {
            throw new InvalidArgumentException(sprintf('Username must be %d char\ acters  or  more',  self::MIN_LENGTH));
        }
    }

    private function assertNotTooLong($username)
    {
        if  (strlen($username)  >  self::MAX_LENGTH)  {
            throw new InvalidArgumentException(sprintf('Username must be %d char\ acters  or  less',  self::MAX_LENGTH));
            
        }
    }

    private function assertValidFormat($username)
    {
        if  (preg_match(self::FORMAT,  $username)  !==  1)  {
            throw new InvalidArgumentException('Invalid username format');
        }
    }
}
```



As you can see in the example above, there are four pre-conditions that must be satisfied in order to build a Username value object:



* Must not be empty
* Must be at least 5 characters
* Must be less than 10 characters
* Must follow a format of alphanumeric characters or underscore



If all the pre-conditions are met, the attribute will be set and the object will be successfully built. Otherwise, a InvalidArgumentException will be raised, execution halted and the client will be shown an error.

Some developers may see this kind of validation as defensive programming. However, here we are not checking that the input is a string, or that nulls are not permitted. We cannot avoid people using our code incorrectly, but we can control the correctness of our domain state.



