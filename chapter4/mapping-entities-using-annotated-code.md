One of the features used to present Doctrine when it was released was that mapping information can be specified using annotated code



> What’s an annotation?
>
> An annotation is a special form of metadata. In PHP is put under source code comments. For example, PHPDocumentor makes use of annotations to build API information or PHPUnit uses some annotations to specify dataProviders or to provide expectations about exceptions thrown by a piece of code



```php
class SumTest extends PHPUnit_Framework_TestCase {
    /**
     * @dataProvider aMethodName
     */
    public function testAddition() {
// ...
    }
}
```



So in order to map the Order entity to the persistence store first of all the source code for the Order should be modified to add the Doctrine annotations

