Sometimes, a UseCase is going to be executed from a Cron job or the command line. As examples, batch processing or some testing command lines to accelerate the development.

While testing this feature using the web or the API, you realize that it would be nice to have a command line to do it, so you don’t have to go through the browser.

If you are using shell scripts files, I suggest you to check the Symfony Console component. What would the code look like?





```php
namespace Idy\Console\Command;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
class VoteIdeaCommand extends Command
{
    protected function configure()
    {
        $this
            ->setName('idea:rate')
            ->setDescription('Rate an idea')
            ->addArgument('id', InputArgument::REQUIRED)
            ->addArgument('rating', InputArgument::REQUIRED)
        ;
    }
    protected function execute(
        InputInterface $input,
        OutputInterface $output
    ) {
        $ideaId = $input->getArgument('id');
        $rating = $input->getArgument('rating');
        $ideaRepository = new RedisIdeaRepository();
        $useCase = new RateIdeaUseCase($ideaRepository);
        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );
        $output->writeln('Done!');
    }
}
```



Again those 3 lines of code. As before, the UseCase and its business logic remain untouched, we are just providing a new delivery mechanism. Congratulations, you’ve discovered the user side hexagon edge.

                                                                                                    There is still a lot to do. As you may have heard, a real craftsman does TDD. We have already started our story so we must be ok with just testing after.



