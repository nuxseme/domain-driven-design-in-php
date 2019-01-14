As a good developer, you decide to divide and conquer the user story, so you’ll start with the first part, “I want to rate an idea”. After that, you will face “the author should be notified by email”. That sounds like a plan.

In terms of business rules, rating an idea is as easy as finding the idea by its identifier in the ideas repository, where all the ideas live, add the rating, recalculate the average and save the idea back. If the idea does not exist or the repository is not available we should throw an exception so we can show an error message, redirect the user or do whatever the business asks us for.

In order to execute this UseCase, we just need the idea identifier and the rating from the user. Two integers that would come from the user request.

Your company web application is dealing with a Zend Framework 1 legacy application. As most of companies, probably some parts of your app may be newer, more SOLID, and others may just be a

big ball of mud. However, you know that it does not matter at all which framework you are using, it’s all about writing clean code that makes maintenance a low cost task for your company.

You’re trying to apply some Agile principles you remember from your last conference, how it was, yeah, I remember “make it work, make it right, make it fast”. After some time working you get something like Listing 1.



```php

class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        // Getting parameters from the request
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');
        // Building database connection
        $db = new Zend_Db_Adapter_Pdo_Mysql(array(
            'host' => 'localhost',
            'username' => 'idy',
            'password' => '',
            'dbname' => 'idy'
        ));
        // Finding the idea in the database
        $sql = 'SELECT * FROM ideas WHERE idea_id = ?';
        $row = $db->fetchRow($sql, $ideaId);
        if (!$row) {
            throw new Exception('Idea does not exist');
        }
        // Building the idea from the database
        $idea = new Idea();
        $idea->setId($row['id']);
        $idea->setTitle($row['title']);
        $idea->setDescription($row['description']);
        $idea->setRating($row['rating']);
        $idea->setVotes($row['votes']);
        $idea->setAuthor($row['email']);
        // Add user rating
        $idea->addRating($rating);
        // Update the idea and save it to the database
        $data = array(
            'votes' => $idea->getVotes(),
            'rating' => $idea->getRating()
        );
        $where['idea_id = ?'] = $ideaId;
        $db->update('ideas', $data, $where);
        // Redirect to view idea page
        $this->redirect('/idea/'.$ideaId);
    }
}
```



I know what readers are thinking: “Who is going to access data directly from the controller? This is a 90’s example!”, ok, ok, you’re right. If you are already using a framework, it’s likely that you are also using an ORM. Maybe done by yourself or any of the existing ones such as Doctrine, Eloquent, ZendDB, etc. If this is the case, you are one step further from those who have some Database connection object but don’t count your chickens before they’re hatched.

For newbies, Listing 1 code just works. However, if you take a closer look at the Controller, you’ll see more than business rules, you’ll also see how your web framework routes a request into your business rules, references to the database or how to connect to it. So close, you see references to your infrastructure.

Infrastructure is the detail that makes your business rules work. Obviously, we need some way to get to them \(API, web, console apps, etc.\) and effectively we need some physical place to store our ideas \(memory, database, NoSQL, etc.\). However, we should be able to exchange any of these pieces with another that behaves in the same way but with different implementations. What about starting with the Database access?

All those Zend\_DB\_Adapter connections \(or straight MySQL commands if that’s your case\) are asking to be promoted to some sort of object that encapsulates fetching and persisting Idea objects. They are begging for being a Repository.



