There are times when an object composed of valid properties, as a whole, can still be deemed invalid. It can be tempting to add this kind of validation to the object itself, but generally this is an anti- pattern. Higher-level validation is likely to change at different times to the object itself. Also it is good practice to separate these responsibilities.

The validation informs the client about any errors that have been found, or collect the results to be reviewed later. Sometimes we do not want to stop the execution at the first sign of trouble.

An abstract and reusable Validator could be something like:



```php

abstract class Validator
{
    private $validationHandler;

    public function construct(ValidationHandler $validationHandler)
    {
        $this->validationHandler = $validationHandler;
    }

    protected function handleError($error)
    {
        $this->validationHandler->handleError($error);
    }

    abstract public function validate();
}
```

As a concrete example, we want to validate an entire Location object, composed of valid Country, City and Postcode value objects. These individual values however, might be in an invalid state at the time of validation. Maybe the city does not form part of the country or maybe the postcode does not follow the city format.

```php

class Location
{
    private $country; private $city; private $postcode;

    public function construct(Country $country, City $city, Postcode $postcode)
    {
        $this->country = $country;
        $this->city = $city;
        $this->postcode = $postcode;
    }

    public function getCountry()
    {
        return $this->country;
    }

    public function getCity()
    {
        
        return $this->city;
    }

    public function getPostcode()
    {
        return $this->postcode;
    }
}

```



The validator checks the state of the Location object in its entirety, analysing the meaning of the relationships between properties:



```php
class LocationValidator extends Validator
{
    private $location;

    public function construct(Location $location, ValidationHandler $validatio\ nHandler)
    {
        parent::construct($validationHandler);
        $this->location = $location;
    }

    public function validate()
    {
        if (!$this->location->getCountry()->hasCity($this->location->getCity()))\{
            $this->handleError('City not found');
        }
    
        if (!$this->location->getCity()->isPostcodeValid($this->location->getPos\ tcode())) {
            $this->handleError('Invalid postcode');
        }
    }
}
```



Once all the properties have been set we are able to validate the entity, most likely after some described process. On the surface it looks as if the Location validates itself, this however is not the case. Location delegates this validation to a concrete validator instance, splitting these two clear responsibilities.



```php

class Location
{
//...

    public function validate(ValidationHandler $validationHandler)
    {
        $validator = new LocationValidator($this, $validationHandler);
        $validator->validate();
    }
}
```



