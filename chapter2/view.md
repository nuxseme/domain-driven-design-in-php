The View is a layer that can both receive and send messages from the Model layer and/or from the

Controller layer. Its main purpose is to represent the Model to the user at the UI level, and refresh the

representation in the UI each time the Model is updated. Generally speaking, the View layer receives

an object, often a Data Transfer Object \(DTO\) instead of instances of the Model layer, gathering all

the needed information to be successfully represented. For PHP there are several template engines

that can help a great deal in separating the Model representation from the Model itself and from the

Controller. The most popular one by far is called Twig². Lets see how the View layer would look

like with Twig

> DTOs instead of Model instances?
>
> This is an old and active topic. Why create a DTO instead of giving an instance of the Model
>
> to the View layer? The main reason and the short answer is, again, Separation of Concerns.
>
> Letting the view inspect and use a Model instance leads to tight coupling between the View
>
> layer and the Model layer. In fact, a change in the Model layer can potentially break all the
>
> views that make use of the changed Model instances.

```php
{% extends "base.html.twig" %}
{% block content %}
{% if errormsg is defined %}
<div class="alert error">{{ errormsg }} </div>
{% else %}
<div class="alert success">Bravo! Post was created successfully!</div>
{% endif %}
<table>
    <thead><tr><th>ID</th><th>TITLE</th><th>ACTIONS</th></tr></thead>
    <tbody>
    {% for post in posts %}
    <tr>
        <td>{{ post.id }}</td>
        <td>{{ post.title }}<?php echo $post['TITLE']; ?></td>
        <td><a href="{{ editPostUrl(post.id) }}">Edit Post</a></td>
    </tr>
    {% endforeach %}
    </tbody>
</table>
{% endblock %}
```

  


Most of the time, when the Model triggers a state change, it also notifies the related Views so that

the UI can get refreshed. In a typical web scenario the synchronization between the Model and its

representations can be a bit tricky because of the client-server nature. In this kind of environments

some JavaScript defined interactions are usually needed to maintain that synchronization. For this

reason, JavaScript MVC frameworks like the ones below have become widely popular in recent years:

• AngularJS³

• EmberJS⁴

• Marionette⁵

• ReactJS⁶

