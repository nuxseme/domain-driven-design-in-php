The Controller layer is responsible for organizing and orchestrating the View and the Model. It

receives messages from the View layer and triggers Model behaviour in order to perform the desired

action. Furthermore, it sends messages to the View in order to display Model representations. Both

operations are performed thanks to the Application Layer, responsible for orchestrating, organizing

and encapsulating domain behaviour.

In terms of a web application in PHP, the Controller usually comprehends a set of classes, which

in order to fulfill their purpose “speak HTTP”. That is, they receive an HTTP request and return an

HTTP response.

```php
class PostsController
{
    public function updateAction(Request $request)
    {
        if ($request->request->has('submit')
            && Validator::validate($request->request->post)
        ) {
            $postService = new PostService();
            try {
                $postService->createPost(
                    $request->request->get('title'),
                    $request->request->get('content')
                );
                $this->addFlash(
                    'notice',
                    'Post has been created successfully!'
                );
            } catch (Exception $e) {
                $this->addFlash(
                    'error',
                    'Unable to create the post!'
                );
            }
        }
        return $this->render('posts/update-result.html.twig');
    }
}
```



