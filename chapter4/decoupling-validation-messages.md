With some minor changes to our existing implementation, we are able to decouple the validation messages from the validator:

```php


class LocationValidationHandler implements ValidationHandler
{
    public function handleCityNotFoundInCountry();

    public function handleInvalidPostcodeForCity();
}

class LocationValidator
{
    private $location;
    private $validationHandler;

    public function construct(Location $location, LocationValidationHandler $v\ alidationHandler)
    {
        $this->location = $location;
        $this->validationHandler = $validationHandler;
    }

    public function validate()
    {
        if (!$this->location->getCountry()->hasCity($this->location->getCity()))\
        {
            $this->validationHandler->handleCityNotFoundInCountry();
        }
        
        if (!$this->location->getCity()->isPostcodeValid($this->location->getPos\ tcode())) {
            $this->validationHandler->handleInvalidPostcodeForCity();
        }
    }
}

```



