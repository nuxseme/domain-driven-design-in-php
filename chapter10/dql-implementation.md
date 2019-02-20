In the case of this repository we will only need the EntityManager to retrieve domain objects directly from the database

对于这个存储库，我们只需要EntityManager直接从数据库检索域对象

```php
namespace Infrastructure\Persistence\Doctrine;

use Doctrine\ORM\EntityManager;
use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;

class DoctrinePostRepository implements PostRepository
{
    protected  $em;

    public function construct(EntityManager $em)
    {
        $this->em  =  $em;
    }

    public function add(Post $aPost)
    {
        $this->em->persist($aPost);
    }

    public function remove(Post $aPost)
    {
        $this->em->remove($aPost);
    }

    public function postOfId(PostId $anId)
    {
        return  $this->em->find('Domain\Model\Post',  $anId);
    }

    public function latestPosts(\DateTime $sinceADate)
    {
        return  $this->em->createQueryBuilder()
            ->select('p')
            ->from('Domain\Model\Post', 'p')
            ->where('p.createdAt > :since')
            ->setParameter(':since', $sinceADate)
            ->getQuery()
            ->getResult();
    }

    public function nextIdentity()
    {
        return new PostId();
    }
}
```



