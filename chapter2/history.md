  
Before the release of PHP version 5, the language did not embrace the Object-Oriented paradigm.

Back in these days, the usual way to write applications was by using procedures and global state.

Concepts like Separation of Concerns, MVC and such were very alien among the PHP community.

The example below, is an application written in this ‘traditional way’, where applications were

composed of many front controllers mixed with HTML code. During this time Infrastructure,

Presentation or UI and Domain layer code was tangled all together

```php
<?php
include__DIR__ . '/bootstrap.php';
$db       = newPDO('mysql:host=localhost;dbname=my_database', 'a_username', '4_p4ssw0\rd',
    [PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8',]);
$errormsg = null;
if (isset($_POST['submit'] && isValid($_POST['post'])) {
    $post = getFrom($_POST['post']);
    $db->beginTransaction();
    try {
        $stm = $db->prepare('INSERT INTO posts (title, content) VALUES (?, ?)');
        $stm->exec([
            $post['title'],
            $post['content']
        ]);
        $db->commit();
    } catch (Exception $e) {
        $db->rollback();
        $errormsg = 'Post could not be created! :(';
    }
}
$stm   = $db->prepare('SELECT id, title, content FROM posts');
$posts = $stm->fetchAll(PDO::FETCH_ASSOC);
?>
<html>
<head></head>
<body>
<?php if (null !== $errormsg): ?>
    <div class="alert error"><?php echo $errormsg; ?></div>
<?php else: ?>
    <div class="alert success">Bravo! Post was created successfully!</div>
<?php endif; ?>
<table>
    <thead>
    <tr>
        <th>ID</th>
        <th>TITLE</th>
        <th>ACTIONS</th>
    </tr>
    </thead>
    <tbody> <?php foreach ($posts as $post): ?>
        <tr>
            <td><?php echo $post['ID']; ?></td>
            <td><?php echo $post['TITLE']; ?></td>
            <td><?php editPostUrl($post['ID']); ?></td>
        </tr>
    <?php endforeach; ?>
    </tbody>
</table>
</body>
</html>
```

  
  


This style of coding is often referred to as the Big Ball of Mud¹. An improvement seen in this style 

however, was to encapsulate the header and the footer of the web page in their own separate files,

which were included in the others. This avoided duplication and favoured reuse





```php
<?php
include __DIR__ . '/bootstrap.php';
$db = new PDO('mysql:host=localhost;dbname=my_database', 'a_username', '4_p4ssw0\
rd', [
    PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8',
]);
$errormsg = null;
if (isset($_POST['submit'] && isValid($_POST['post'])) {
    $post = getFrom($_POST['post']);
    $db->beginTransaction();
    try {
        $stm = $db->prepare('INSERT INTO posts (title, content) VALUES (?, ?)');
        $stm->exec([
            $post['title'],
            $post['content']
        ]);
        $db->commit();
    } catch (Exception $e) {
        $db->rollback();
        $errormsg = 'Post could not be created! :(';
    }
}
$stm = $db->prepare('SELECT id, title, content FROM posts');
$posts = $stm->fetchAll(PDO::FETCH_ASSOC);
?>
<?php include __DIR__ . '/header.php'; ?>
<?php if (null !== $errormsg): ?>
    <div class="alert error"><?php echo $errormsg; ?></div>
<?php else: ?>
    <div class="alert success">Bravo! Post was created successfully!</div>
<?php endif; ?>
<table>
    <thead><tr><th>ID</th><th>TITLE</th><th>ACTIONS</th></tr></thead>
    <tbody>
    <?php foreach ($posts as $post): ?>
    <tr>
        <td><?php echo $post['ID']; ?></td>

        <td><?php echo $post['TITLE']; ?></td>
        <td><?php editPostUrl($post['ID']); ?></td>
    </tr>
    <?php endforeach; ?>
    </tbody>
</table>
<?php include __DIR__ . '/footer.php'; ?>
```



Nowadays, and although it is highly discouraged, there are still applications that use this procedural

way of coding. The main disadvantage of this style of architecture is that there is no real separation

of concerns - maintenance and cost of change increases drastically in relation to other well-known

and proven architectures.

