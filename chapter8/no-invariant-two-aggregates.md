As a User, I want to make a Wish”. We will go into Application Services in following chapters, but for now, let’s check different approaches for making a Wish. We think that the first approach for the novice will be something similar to the following code.

```php
class MakeWishService
{
    private $wishRepository;

    public function construct(WishRepository $wishRepository)
    {
        $this->wishRepository = $wishRepository;
    }

    public function execute(MakeWishRequest $request)
    {
        $userId = $request->userId();
        $address   =  $request->address();
        $content   =  $request->content();

        $wish = new Wish(
            $this->wishRepository->nextIdentity(),
            new UserId($userId),
            $address,
            $content
        );

        $this->wishRepository->add($wish);
    }
}
```

How you see this code? It is probably the most performing code possible. Behind the scenes you can almost see the INSERT statement. It’s the minimum number of operations for such a use case, one, so good job. With the current implementation you can create as many wishes as you want, following the business logic, so good job again.

However, there may be just one potential issue, you can create wishes for a user that may no exist in your Domain. That is a problem. Indeed, it doesn’t really matter if you will use a Relational database or a NoSQL one, before persisting anything, with this approach, you can create a Wish without its corresponding User in memory.

Indeed, it’s a broken business logic. I know, I know, you can fix that using a foreign key in the database, from wish\(user\_id\) to user\(id\). Correct, but, what happen if we are not using a database with foreign keys, even more, what happen if is a NoSQL database, such as Redis or ElasticSearch?

So, if we want to fix this issue so the same code can work properly in different infrastructures, we need to check if the user exists. Probably, the easiest approach could be in the same Application Service, couldn’t be?

```php

class MakeWishService
{
//...

    public function execute(MakeWishRequest $request)
    {
        $userId = $request->userId();
        $address   =  $request->address();
        $content   =  $request->content();

        $user = $this->userRepository->ofId(new UserId($userId));
        if (null === $user) {
            throw new UserDoesNotExistException();
        }

        $wish = new Wish(
            $this->wishRepository->nextIdentity(),
            $user->id(),
            $address,
            $content
        );

        $this->wishRepository->add($wish);
    }
}
```

That could make the trick, but what’s the problem about doing the check in the Application Service? There is no protection about that in our Domain, anyone, inside a Domain Service, any part of the infrastructure, etc. could do something such as the following code.



```php
// Somewhere in your domain
$nonExistingUserId = new UserId('non-existing-user-id');
$wish = new Wish(
    $this->wishRepository->nextIdentity(),
    $nonExistingUserId,
    $address,
    $content
);

```



If you have already read the Factories chapter you have got the solution too. Factories help us keeping the business invariants and that’s exactly what we need here. There’s an implicit invariant saying that we are not allowed to make a wish without a valid user. Let’s see how a factory can help us.

```php
class MakeWishService
{
    private $userRepository;
    private $wishRepository;

    public function construct( UserRepository $userRepository, WishRepository $wishRepository
    )
    {
        $this->userRepository   = $userRepository;
        $this->wishRepository    =   $wishRepository;
    }

    public function execute(MakeWishRequest $request)
    {
        $userId = $request->userId();
        $address = $request->address();
        $content = $request->content();

        $user = $this->userRepository->ofId(new UserId($userId));
        if (null === $user) {
            throw new UserDoesNotExistException();
        }

        $wish = $user->makeWish(
            $this->wishRepository->nextIdentity(),
            $address,
            $content
        );

        $this->wishRepository->add($wish);
    }
}
```



As you can see, Users make wishes, so our code does. makeWish is a factory method for building

Wishes. The method returns a new wish build with the UserId.



```
class User
{
// ...

    /**
     * @return Wish
     */
    public function makeWish(WishId $wishId, $address, $content)
    {
        return  new Wish(
            $wishId,
            $this->id(),
            $address,
            $content
        );
    }

// ...
}

```



Why are we returning a Wish and not just adding the new Wish to an internal collection as we would do with Doctrine probably?



To sum up, in this scenario, User and Wish do not conform an aggregate. Each Entity has its own Repository and they are linked using UserId. Getting all the wishes of a User can be performed by a finder in the wishes Repository.





