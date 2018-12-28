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



view层可以接受和发送来自model和controller的数据。它的主要目的是展示模型给到UI,模型改变了会刷新数据。通常来讲，在model层和view会使用DTO来流转数据。在收集完所有必要的数据之后才展示。在php领域有很多渲染模板的引擎，如流行的Twig



> 为什么用DTO取代Model实例
>
> 古老而活跃的话题。简而言之，解耦。模型的改变可能破坏所有依赖模型实例的views

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

大多数时候，当模型触发状态更改时，它还会通知相关的视图UI可以刷新。在一个典型的web场景中，模型与其之间的同步由于客户机-服务器的特性，表示可能有点棘手。在这种环境中通常需要一些JavaScript定义的交互来维护同步。JavaScript MVC框架，像下面这些，在最近几年变得非常流行:

• AngularJS³

• EmberJS⁴

• Marionette⁵

• ReactJS⁶



---

* DTO  这个在php用的没那么正式 通常返回json，需要前端根据获取的参数和字段处理。接口的响应数据是可以事先定义的









