Michael Feathers introduced a definition of legacy code as code without tests. You don’t want your code to be legacy just born, do you?

In order to unit test this UseCase object, you decide to start with the easiest part, what happens if the repository is not available? How can we generate such behavior? Do we stop our Redis server while running the unit tests? No. We need to have an object that has such behavior. Let’s use a mock object in Listing



```php
class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function whenRepositoryNotAvailableAnExceptionShouldBeThrown()
    {
        $this->setExpectedException('NotAvailableRepositoryException');
        $ideaRepository = new NotAvailableRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $useCase->execute(
            new RateIdeaRequest(1, 5)
        );
    }
}
class NotAvailableRepository implements IdeaRepository
{
    public function find($id)
    {
        throw new NotAvailableException();
    }
    public function update(Idea $idea)
    {
        throw new NotAvailableException();
    }
}
```





Nice. NotAvailableRepository has the behavior that we need and we can use it with RateIdeaUse- Case because it implements IdeaRepository interface.

Next case to test is what happens if the idea is not in the repository. Listing 10 shows the code.





```php
class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
    // ...
    /**
     * @test
     */
    public function whenIdeaDoesNotExistAnExceptionShouldBeThrown()
    {
        $this->setExpectedException('IdeaDoesNotExistException');
        $ideaRepository = new EmptyIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $useCase->execute(
            new RateIdeaRequest(1, 5)
        );
    }
}
class EmptyIdeaRepository implements IdeaRepository
{
    public function find($id)
    {
        return null;
    }
    public function update(Idea $idea)
    {
    }
}
```





Here, we use the same strategy but with an EmptyIdeaRepository. It also implements the same interface but the implementation always returns null regardless which identifier the find method receives.

Why are we testing these cases?, remember Kent Beck’s words: “Test everything that could possibly break”.

Let’s carry on with the rest of the feature. We need to check a special case that is related with having a read available repository where we cannot write to. Solution can be found in Listing





```php

class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
// ...
    /**
     * @test
     */
    public function whenUpdatingInReadOnlyAnIdeaAnExceptionShouldBeThrown()
    {
        $this->setExpectedException('NotAvailableRepositoryException');
        $ideaRepository = new WriteNotAvailableRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute(
            new RateIdeaRequest(1, 5)
        );
    }
}
class WriteNotAvailableRepository implements IdeaRepository
{
    public function find($id)
    {
        $idea = new Idea();
        $idea->setId(1);
        $idea->setTitle('Subscribe to php[architect]');
        $idea->setDescription('Just buy it!');
        $idea->setRating(5);
        $idea->setVotes(10);
        $idea->setAuthor('hi@carlos.io');
        return $idea;
    }
    public function update(Idea $idea)
    {
        throw new NotAvailableException();
    }
}
```





Ok, now the key part of the feature is still remaining. We have different ways of testing this, we can write our own mock or use a mocking framework such as Mockery or Prophecy. Let’s choose the first one. Another interesting exercise would be to write this example and the previous ones using one of these frameworks.







```php

class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
// ...
    /**
     * @test
     */
    public function whenRatingAnIdeaNewRatingShouldBeAddedAnIdeaUpdated()
    {
        $ideaRepository = new OneIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute(
            new RateIdeaRequest(1, 5)
        );
        $this->assertSame(5, $response->idea->getRating());
        $this->assertTrue($ideaRepository->updateCalled);
    }
}
class OneIdeaRepository implements IdeaRepository
{
    public $updateCalled = false;
    public function find($id)
    {
        $idea = new Idea();
        $idea->setId(1);
        $idea->setTitle('Subscribe to php[architect]');
        $idea->setDescription('Just buy it!');
        $idea->setRating(5);
        $idea->setVotes(10);
        $idea->setAuthor('hi@carlos.io');
        return $idea;
    }
    public function update(Idea $idea)
    {
        $this->updateCalled = true;
    }
}
```



Bam! 100% Coverage for the UseCase. Maybe, next time we can do it using TDD so the test will come first. However, testing this feature was really easy because of the way decoupling is promoted in this architecture.

Maybe you are wondering about this:



`$this->updateCalled = true;`



We need a way to guarantee that the update method has been called during the UseCase execution. This does the trick. This test double object is called a spy, mocks cousin.

When to use mocks? As a general rule, use mocks when crossing boundaries. In this case, we need mocks because we are crossing from the domain to the persistence boundary.

What about testing the infrastructure?



