The read model, also known as the Query Model, is a pure denormalized data model lifted from domain concerns. In fact, with CQRS all the read concerns are treated as reporting processes, an infrastructure concern. In general, when using CQRS, the read model is subject to the needs of the UI and how complex the views compounding the UI are. In a situation where the the read model is defined in terms of relational databases, the simplest approach would be to set one-to-one relationships between database tables and UI views. These database tables / UI views will be updated using write model projections triggered from the domain events published by the write side.

读模型，也称为查询模型，是从领域关注点中提取的纯非规范化数据模型。事实上，使用CQRS，所有读取关注点都被视为报告流程，一个基础架构关注点。通常，在使用CQRS时，读取模型取决于UI的需求以及组成UI的视图有多复杂。在使用关系数据库定义读模型的情况下，最简单的方法是在数据库表和UI视图之间设置一对一的关系。这些数据库表/ UI视图将使用写方发布的域事件触发的写模型投影来更新。



-- Definition of a UI view of a single post with its comments

```php
CREATE TABLE single_post_with_comments (
id INTEGER NOT NULL,
post_id INTEGER NOT NULL,
post_title VARCHAR(100) NOT NULL,
post_content TEXT NOT NULL,
post_created_at DATETIME NOT NULL,
comment_content TEXT NOT NULL
);
```

-- Set up some data

```php
INSERT INTOsingle_post_with_comments(1,1, "Layered architecture", "Lorem ipsum\
dolor sit amet, ...", NOW(), "Lorem ipsum dolor sit amet, ...");
INSERT INTOsingle_post_with_comments(2,1, "Layered architecture", "Lorem ipsum\
dolor sit amet, ...", NOW(), "Lorem ipsum dolor sit amet, ...");
INSERT INTOsingle_post_with_comments(3,2, "Hexagonal architecture", "Lorem ips\
um dolor sit amet, ...", NOW(), "Lorem ipsum dolor sit amet, ...");
INSERT INTOsingle_post_with_comments(4,2, "Hexagonal architecture", "Lorem ips\
um dolor sit amet, ...", NOW(), "Lorem ipsum dolor sit amet, ...");
INSERT INTOsingle_post_with_comments(5,3, "Command - Query Responsability Segg\
regation", "Lorem ipsum dolor sit amet, ...", NOW(), "Lorem ipsum dolor sit amet\
, ...");
INSERT INTOsingle_post_with_comments(6,3, "Command - Query Responsability Segg\
regation", "Lorem ipsum dolor sit amet, ...", NOW(), "Lorem ipsum dolor sit amet\
, ...");
```

-- Query it

```php
SELECT*FROM single_post_with_comments WHERE post_id = 1;
```

An important feature of this architectural style is that the read model should be completely disposable since the true state of the application is handled by the write model. This means the read model can be removed and recreated when needed using write model projections. Here we can see some examples of possible views within a blog application

这种体系结构风格的一个重要特性是，读模型应该是完全可丢弃的，因为应用程序的真实状态是由写模型处理的。这意味着可以在需要时使用写模型投影删除和重新创建读模型。在这里，我们可以看到博客应用程序中可能视图的一些示例



```php
SELECT * FROM posts_grouped_by_month_and_year ORDER BY month DESC, year ASC;
SELECT*FROM posts_by_tags WHERE tag ="ddd";
SELECT*FROM posts_by_author WHERE author_id = 1;
```

It is important to point out that CQRS does not constrain the definition and implementation of the read model to a relational database. It depends exclusively on the needs of the application being built. It could be a relational database, a document-oriented database, a key-value store or whatever best suits the needs of your application. Following the blog post application, we will use Elasticsearch⁹ \(a document-oriented database\) to implement a read model.

需要指出的是，CQRS没有将读模型的定义和实现限制到关系数据库。它完全取决于正在构建的应用程序的需要。它可以是关系数据库、面向文档的数据库、键值存储或任何最适合应用程序需要的东西。博客应用程序后,我们将使用Elasticsearch⁹\(面向文档的数据库\)来实现阅读模式。



```
class PostsController
{
    public function listAction()
    {
        $client = new \Elasticsearch\ClientBuilder::create()->build();
$response = $client->search([
    'index' => 'blog-engine',
    'type'  => 'posts',
    'body'  => [
        'sort' => [
            'created_at' => ['order' => 'desc']
        ]
    ]
    ]);
    return [
        'posts' => $response
    ];
    }
}
```

The read model code has been drastically simplified to a query to an Elasticsearch index. This reveals that the read model does not really need an object-relational mapper as doing so might be an overkill. However, the write model might benefit from the use of an object-relational mapper as they allow you to organize and structure the read model according to the needs of the application.

读取的模型代码被极大地简化为对一个Elasticsearch索引的查询。这表明，读取模型实际上并不需要对象关系映射器，因为这样做可能有些过头。然而，写模型可能受益于对象关系映射器的使用，因为它们允许您根据应用程序的需要组织和构造读模型。



---

1. ...



