Whether there is a change in the business rules or in the infrastructure, we must edit the same piece of code. Believe me, in CS, you don’t want many people touching the same piece of code for different reasons. Try to make your functions do one and just one thing so it’s less probable having people messing around with the same piece of code. You can learn more about this by having a look at the Single Responsibility Principle \(SRP\). For more information about this principle: http:

//www.objectmentor.com/resources/articles/srp.pdf

Listing 1 is clearly this case. If we want to move to Redis or add the author notification feature, you’ll have to update the rateAction method. Chances to affect aspects of the rateAction not related with

the one updating are high. Listing 1 code is fragile. If it is common in your team to hear “If it works, don’t touch it”, SRP is missing.

So, we must decouple our code and encapsulate the responsibility for dealing with fetching and persisting ideas into another object. The best way, as explained before, is using a Repository. Challenged accepted! Let’s see the results in Listing 2.





```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');
        $ideaRepository = new IdeaRepository();
        $idea = $ideaRepository->find($ideaId);
        if (!$idea) {
            throw new Exception('Idea does not exist');
        }
        $idea->addRating($rating);
        $ideaRepository->update($idea);
        $this->redirect('/idea/'.$ideaId);
    }
}
class IdeaRepository
{
    private $client;
    public function __construct()
    {
        $this->client = new Zend_Db_Adapter_Pdo_Mysql(array(
            'host' => 'localhost',
            'username' => 'idy',
            'password' => '',
            'dbname' => 'idy'
        ));
    }
    public function find($id)
    {
        $sql = 'SELECT * FROM ideas WHERE idea_id = ?';
        $row = $this->client->fetchRow($sql, $id);
        if (!$row) {
            return null;
        }
        $idea = new Idea();
        $idea->setId($row['id']);
        $idea->setTitle($row['title']);
        $idea->setDescription($row['description']);
        $idea->setRating($row['rating']);
        $idea->setVotes($row['votes']);
        $idea->setAuthor($row['email']);
        return $idea;
    }
    public function update(Idea $idea)
    {
        $data = array(
            'title' => $idea->getTitle(),
            'description' => $idea->getDescription(),
            'rating' => $idea->getRating(),
            'votes' => $idea->getVotes(),
            'email' => $idea->getAuthor(),
        );
        $where = array('idea_id = ?' => $idea->getId());
        $this->client->update('ideas', $data, $where);
    }
}
```



The result is nicer. The rateAction of the IdeaController is more understandable. When read, it talks about business rules. IdeaRepository is a business concept. When talking with business guys, they understand what an IdeaRepository is: A place where I put Ideas and get them.

A Repository “mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.” as found in Martin Fowler’s pattern catalog.

If you are already using an ORM such as Doctrine, your current repositories extend from an EntityRepository. If you need to get one of those repositories, you ask Doctrine EntityManager to do the job. The resulting code would be almost the same, with an extra access to the EntityManager in the controller action to get the IdeaRepository.

At this point, we can see in the landscape one of the edges of our hexagon, the persistence edge. However, this side is not well drawn, there is still some relationship between what an IdeaRepository is and how it’s implemented.

In order to make an effective separation between our application boundary and the infrastructure boundary we need an additional step. We need to explicitly decouple behavior from implementation using some sort of interface.



