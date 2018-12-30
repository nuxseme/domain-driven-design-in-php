The Controller layer is responsible for organizing and orchestrating the View and the Model. It receives messages from the View layer and triggers Model behaviour in order to perform the desired action. Furthermore, it sends messages to the View in order to display Model representations. Both operations are performed thanks to the Application Layer, responsible for orchestrating, organizing and encapsulating domain behaviour. In terms of a web application in PHP, the Controller usually comprehends a set of classes, which in order to fulfill their purpose “speak HTTP”. That is, they receive an HTTP request and return an HTTP response.

> 控制层组织和协调view 和 model。接受view层的消息触发modle的行为。同时发送消息到view层用于model层的展示。
>
> 应用层组织协调封装领域行为。对于PHP中的web应用程序，控制器通常包含一组类，这些类是为了实现他们的目的，“ 跟 HTTP 沟通”。接收HTTP请求并返回HTTP响应

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

---

* 接受请求
* 调用service  基础组件  工具类 
* 返回数据



