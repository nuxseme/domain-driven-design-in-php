```php
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



