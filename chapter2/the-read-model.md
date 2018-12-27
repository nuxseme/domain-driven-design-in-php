The read model, also known as the Query Model, is a pure denormalized data model lifted from

domain concerns. In fact, with CQRS all the read concerns are treated as reporting processes, an

infrastructure concern. In general, when using CQRS, the read model is subject to the needs of the

UI and how complex the views compounding the UI are.

In a situation where the the read model is defined in terms of relational databases, the simplest

approach would be to set one-to-one relationships between database tables and UI views. These

database tables / UI views will be updated using write model projections triggered from the domain

events published by the write side.

-- Definition of a UI view of a single post with its comments

CREATE TABLE single\_post\_with\_comments \(

id INTEGER NOT NULL,

post\_id INTEGER NOT NULL,

post\_title VARCHAR\(100\) NOT NULL,

post\_content TEXT NOT NULL,

post\_created\_at DATETIME NOT NULL,

comment\_content TEXT NOT NULL

\);

-- Set up some data

INSERT INTOsingle\_post\_with\_comments\(1,1, "Layered architecture", "Lorem ipsum\

dolor sit amet, ...", NOW\(\), "Lorem ipsum dolor sit amet, ..."\);

INSERT INTOsingle\_post\_with\_comments\(2,1, "Layered architecture", "Lorem ipsum\

dolor sit amet, ...", NOW\(\), "Lorem ipsum dolor sit amet, ..."\);

INSERT INTOsingle\_post\_with\_comments\(3,2, "Hexagonal architecture", "Lorem ips\

um dolor sit amet, ...", NOW\(\), "Lorem ipsum dolor sit amet, ..."\);

INSERT INTOsingle\_post\_with\_comments\(4,2, "Hexagonal architecture", "Lorem ips\

um dolor sit amet, ...", NOW\(\), "Lorem ipsum dolor sit amet, ..."\);

INSERT INTOsingle\_post\_with\_comments\(5,3, "Command - Query Responsability Segg\

regation", "Lorem ipsum dolor sit amet, ...", NOW\(\), "Lorem ipsum dolor sit amet\

, ..."\);

INSERT INTOsingle\_post\_with\_comments\(6,3, "Command - Query Responsability Segg\

regation", "Lorem ipsum dolor sit amet, ...", NOW\(\), "Lorem ipsum dolor sit amet\

, ..."\);

-- Query it

SELECT\*FROM single\_post\_with\_comments WHERE post\_id = 1;

An important feature of this architectural style is that the read model should be completely

disposable since the true state of the application is handled by the write model. This means the

read model can be removed and recreated when needed using write model projections.

Here we can see some examples of possible views within a blog application

SELECT \* FROM posts\_grouped\_by\_month\_and\_year ORDER BY month DESC, year ASC;

SELECT\*FROM posts\_by\_tags WHERE tag ="ddd";

SELECT\*FROM posts\_by\_author WHERE author\_id = 1;

It is important to point out that CQRS does not constrain the definition and implementation of the

read model to a relational database. It depends exclusively on the needs of the application being

built. It could be a relational database, a document-oriented database, a key-value store or whatever

best suits the needs of your application.

Following the blog post application, we will use Elasticsearch⁹ \(a document-oriented database\) to

implement a read model.

class PostsController

{

public function listAction\(\)

{

$client=new \Elasticsearch\ClientBuilder::create\(\)-&gt;build\(\);

$response = $client-&gt;search\(\[

'index' =&gt; 'blog-engine',

'type' =&gt; 'posts',

'body' =&gt;\[

'sort' =&gt;\[

'created\_at' =&gt;\['order' =&gt; 'desc'\]

\]

\]

\]\);

return \[

'posts'=&gt; $response

\];

}

}

The read model code has been drastically simplified to a query to an Elasticsearch index. This reveals

that the read model does not really need an object-relational mapper as doing so might be an overkill.

However, the write model might benefit from the use of an object-relational mapper as they allow

you to organize and structure the read model according to the needs of the application.





