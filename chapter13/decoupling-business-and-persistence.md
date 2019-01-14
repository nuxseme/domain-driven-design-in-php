Have you ever experienced the situation when you start talking to your Product Owner, Business Analyst or Project Manager about your issues with the Database? Can you remember their faces when explaining how to persist and fetch an object? They had no idea what you were talking about.

The truth is that they don’t care, but that’s ok. If you decide to store the ideas in a MySQL server, Redis or SQLite it is your problem, not theirs. Remember, from a business standpoint, your infrastructure is a detail. Business rules are not going to change whether you use Symfony or Zend Framework, MySQL or PostgreSQL, REST or SOAP, etc.

That’s why it’s important to decouple our IdeaRepository from its implementation. The easiest way is to use a proper interface. How can we achieve that? Let’s take a look at Listing 3.





```php

class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');
        $ideaRepository = new MySQLIdeaRepository();
        $idea = $ideaRepository->find($ideaId);
        if (!$idea) {
            throw new Exception('Idea does not exist');
        }
        $idea->addRating($rating);
        $ideaRepository->update($idea);
        $this->redirect('/idea/'.$ideaId);
    }
}
interface IdeaRepository
{
    /**
     * @param int $id
     * @return null|Idea
     */
    public function find($id);
    /**
     * @param Idea $idea
     */
    public function update(Idea $idea);
}
class MySQLIdeaRepository implements IdeaRepository
{
// ...
}
```



Easy, isn’t it? We have extracted the IdeaRepository behaviour into an interface, renamed the IdeaRepository into MySQLIdeaRepository and updated the rateAction to use our MySQLIdeaRepos- itory. But what’s the benefit?

We can now exchange the repository used in the controller with any implementing the same interface. So, let’s try a different implementation.



