

Our application is a huge success and now it’s time to monetize. We want new users to have a maximum of three wishes available. As a User, if you want to have more wishes you’ll probably have to pay for a premium account. Let’s see how we could change our code to follow the new business rule about the maximum number of wishes \(do not consider the premium user\).





```
class MakeWishService
{
// ...

    public function execute(MakeWishRequest $request)
    {
        $userId   =  $request->userId();
        $address   =   $request->email();
        $content = $request->content();

        $count = $this->wishRepository->numberOfWishesByUserId($userId);
        if ($count > 3) {
            throw new MaxNumberOfWishesExceededException();
        }

        $wish  =  new  Wish(
            $this->wishRepository->nextIdentity(),
            new UserId($userId),
            $address,
            $content
        );

        $this->wishRepository->add($wish);
    }
}
```



That was easy, wasn’t it? Probably too much easy. We see here different problems. First one is that Application Services should not include such business logic. They must coordinate, but not contain business logic. Probably, a better place is to put them into the User. We can have more control about the relation between User and Wish. However, for the problem explained here, the code works.



The second problem is about the code itself. It does not work under race conditions. So, it nos acceptable. Forget about Domain-Driven Design, what’s the problem with this code in heavy traffic? Think for a minute. Could be possible to break the rule of a User to have more than three wishes? Why your QA running her freaky tests is going to be super happy?

Your QA tries two times in a calm way and ends up with a user with two wishes. Nice. Your QA wants blood. Imagine for a second that opens two tabs in her browser, fills both two forms and submits the two buttons at the same time. Suddenly, after two requests, the user ends up with four wishes in the database. What happened?

Think as a debugger, consider both requests getting at the if \($count &gt; 3\) { line at the same time. Both of the requests will evaluate to false, because the user has just two wishes. So, both requests will create the Wish and both of the request will add it into the database. Ouch! four wishes in a User.

We know what you’re thinking. It’s because we missed to put everything into a transaction. Well, imagine that the same previous code is put inside a transaction block, you will see how to do it properly in the Application Services chapter. Internally, let’s check what’s happening with your database.



```php
START TRANSACTION;
SELECT  @a:=COUNT(*)  FROM  wishes  WHERE  user_id  =  'e3bb5953-5dd2-4204-a8e8-3449ea8\ 88c40';
-- @a is 2
INSERT INTO wishes(id, user_id, address, message) VALUES ('7b81e576-d7dc-4736-a5\ 59-340961b7102a', 'e3bb5953-5dd2-4204-a8e8-3449ea888c40', 'mom@myfamily.com', 'I\ always love you!');
-- Ok!
SELECT  @a:=COUNT(*)  FROM  wishes;
-- @a is 3
COMMIT;
```



If you take this SQL block and execute it line by line in two different connections \(with different UUIDs for the new wishes\), you will see how the wishes table is going to have 4 rows at the end of both executions. So, it looks like is not about the transaction.

How could we fix this issue? Probably, you may have heart about Pessimistic Concurrency and Optimistic Concurrency in persistence mechanisms let’s explore them.



