Although the implementation described above clearly defines the separation of concerns, we are required to repeat the password verification algorithm every time we wish to implement a new encryption mechanism.

An alternative means of solving this problem, aiding code reuse, is by separating out these two responsibilities. We could instead extract the password encryption logic out into a specialised class, using the Strategy Pattern² for all defined encryption algorithms. This leaves the design open for extension and closed for modification.



```php

class SignIn
{
    private $userRepository;
    private $passwordEncryption;

    public  function      construct( UserRepository $userRepository, PasswordEncryption $passwordEncryption
    ) {
        $this->userRepository = $userRepository;
        $this->passwordEncryption = $passwordEncryption;
    }

    public  function  execute($aUsername,  $aPassword)
    {
        if  (!$this->userRepository->has($aUsername))  {
            throw new InvalidArgumentException(
                sprintf('The user "%s" does not exist.', $aUsername)
            );
        }

        $aUser = $this->userRepository->byUsername($aUsername);

        if ($this->isPasswordInvalidFor($aUser, $aPassword)) {
            throw new BadCredentialsException($aUser, $aPassword);
        }

        return $aUser;
    }

    private function isPasswordInvalidFor(User $aUser, $plainPassword)
    {
        return !$this->passwordEncryption->verify(
            $plainPassword,
            $aUser->hash()
        );
    }
}

interface PasswordEncryption
{
    /**
    @param string $password
    @param string $hash
    @return boolean
     */
    public function verify($plainPassword, $hash);
}
```

Defining different encryption strategies is as easy as implementing the PasswordEncryption

interface.

```php
class BasicPasswordEncryption implements PasswordEncryption
{
    public function verify($plainPassword, $hash)
    {
        return password_verify($plainPassword, $hash);
    }
}

class Md5PasswordEncryption implements PasswordEncryption
{
    const SALT = 'S0m3S4lT';

    public function verify($plainPassword, $hash)
    {
        return $hash === $this->calculateHash($plainPassword);
    }

    private function calculateHash($plainPassword)
    {
        return md5($plainPassword . '_' . $this->salt());
    }

    private function salt()
    {
        return  md5(self::SALT);
    }
}
```



