In order to be sure that the repository will work in production, you will need to test its implementa- tion. To do this we have to test the boundaries of the system making sure that our expectations are correct.

In the case of a Doctrine test, the setup will be a little bit more sophisticated

为了确保存储库能够在生产环境中工作，您需要测试它的实现。要做到这一点，我们必须测试系统的边界，以确保我们的期望是正确的。

在原则测试的情况下，设置会稍微复杂一些

```php
use Doctrine\DBAL\Types\Type;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Tools;
use Domain\Model\Post;
class DoctrinePostRepositoryTest extends \PHPUnit_Framework_TestCase
{
    private $postRepository;
    public function setUp()
    {
        $this->postRepository = $this->createPostRepository();
    }
    private function createPostRepository()
    {
        $this->addCustomTypes();
        $em = $this->initEntityManager();
        $this->initSchema($em);
        return new PrecociousDoctrinePostRepository($em);
    }
    private function addCustomTypes()
    {
        if (!Type::hasType('post_id')) {
            Type::addType('post_id', 'Infrastructure\Persistence\Doctrine\Types\PostIdType');
        }
        if (!Type::hasType('body')) {
            Type::addType('body', 'Infrastructure\Persistence\Doctrine\Types\BodyType');
        }
    }
    protected function initEntityManager()
    {
        return EntityManager::create(
            ['url' => 'sqlite:///:memory:'],
            Tools\Setup::createXMLMetadataConfiguration(
                ['/Path/To/Infrastructure/Persistence/Doctrine/Mapping'],
                $devMode = true
            )
        );
    }
    private function initSchema(EntityManager $em)
    {
        $tool = new Tools\SchemaTool($em);
        $tool->createSchema([
            $em->getClassMetadata('Domain\Model\Post')
        ]);
    }
}
class PrecociousDoctrinePostRepository extends DoctrinePostRepository
{
    public function persist(Post $aPost)
    {
        parent::persist($aPost);
        $this->em->flush();
    }
    public function remove(Post $aPost)
    {
        parent::remove($aPost);
        $this->em->flush();
    }
}
```

Once we have this environment setup, we can now continue to test the Repository’s behaviour

```php
class DoctrinePostRepositoryTest extends \PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itShouldRemovePost()
    {
        $post = $this->persistPost('irrelevant body');
        $this->postRepository->remove($post);
        $this->assertPostExist($post->id());
    }
    private function assertPostExist($id)
    {
        $result = $this->postRepository->postOfId($id);
        $this->assertNull($result);
    }
    private function persistPost($body, \DateTime $createdAt = null)
    {
        $this->postRepository->add(
            $post = new Post(
                $this->postRepository->nextIdentity(),
                new Body($body),
                $createdAt
            )
        );
        return $post;
    }
}
```

Following our assertion made earlier, if we save a Post, we expect to find it in the exact same state.

Now we can move on to test finding the latest posts specifying a given date

按照前面的断言，如果保存一个Post，我们希望它处于完全相同的状态。

现在我们可以继续测试指定给定日期的最新帖子

```php
class DoctrinePostRepositoryTest extends \PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itShouldFetchLatestPosts()
    {
        $this->persistPost('a year ago', new \DateTime('-1 year'));
        $this->persistPost('a month ago', new \DateTime('-1 month'));
        $this->persistPost('few hours ago', new \DateTime('-3 hours'));
        $this->persistPost('few minutes ago', new \DateTime('-2 minutes'));
        $posts = $this->postRepository->latestPosts(new \DateTime('-24 hours'));
        $this->assertCount(2, $posts);
        $this->assertEquals('few hours ago', $posts[0]->body()->content());
        $this->assertEquals('few minutes ago', $posts[1]->body()->content());
    }
}
```



