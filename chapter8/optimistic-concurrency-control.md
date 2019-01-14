This approach assumes that conflicts are unlikely to happen and doesn’t block operations from being attempted. However, if the underlying data has been modified between reading and writing, the update will fail. It is then up to the application to decide how it should resolve the conflict. For instance, it could reattempt the update, using the fresh data, or it could report the situation to the user.

Could we use any of these strategies for this use case? Mmmm, we don’t think so. Because we’re making a new wish,





```php

class User
{
// ...

    /**
     * @return void
     */
    public function makeWish(WishId $wishId, $address, $content)
    {
        $this->wishes[] = new Wish(
            $wishId,
            $this->id(),
            $address,
            $content
        );
    }

// ...
}
```



