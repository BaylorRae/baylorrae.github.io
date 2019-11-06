---
layout: post
title: The Basics of Creating a CMS with PHP
date: 2012-03-14 08:00:27.000000000 -05:00
categories:
- PHP
- Tutorials
tags:
- application architecture
- cms
- content management system
- pdo
- php
- tutorial
meta:
  dsq_thread_id: '275 http://baylorrae.com/?p=275'
redirect_from: /blog/2012/03/14/the-basics-of-creating-a-cms-with-php/
---

When people want to build a website for someone, they usually need to create a
Content Management System (CMS) that allows the client to manage their own
content. And while there are thousands of prebuilt CMSes that can get the job
done, it can be very helpful to learn how to build one yourself.

### What a CMS should do

They don't have to be extremely difficult and should only do as much as needed.
If your grandmother wants a website to sell pies, then she only needs a way to
add, update and remove various flavors.

### How a CMS should work

There are two popular methods (that I'm aware of) for creating content
management systems. The first uses a separate area for all administrative
controls. This is my personal choice because it allows the admin area to scale
and provide more features.

The second is what I would call an inline approach, where all backend controls
are done inside the same frontend website. Implementing this approach it helps
keep your code DRY (don't repeat yourself) by keeping you from writing the code
to display pages and content twice.

Each of of these methods have their benefits and detriments. By separating the
administrative controls you have to rewrite most of the code to display the
content again. But, when using an inline approach it's difficult to add a page
to statistically show popular products (or pies). As a result, every website
will need a system tailored to improve the usability for the client.

### Getting Started

The CMS we will be creating will be more of an inline approach. It will be a
very basic site that allows me to store a list of my favorite websites.

Therefore we only need three pages.

1. A home page that shows all of my favorite websites.
2. A form to that lets us create and edit websites.
3. A page that deletes a website from our list.

This is the file structure that I've chosen for our CMS. We will be creating
more files later on, but this should be plenty to get you started.

```text
.
├── global.php // will store our connection to the database
├── index.php // will list our favorite websites
└── models/
    └── website.php // will be a class to store each website into an object

1 directory, 3 files
```

### Creating our Database

We won't be creating a large database for this tutorial. We only want to provide
an easy way to manage our favorite websites.

```sql
CREATE TABLE  `cmsTutorial`.`websites` (
`id` INT NOT NULL AUTO_INCREMENT PRIMARY KEY ,
`url` VARCHAR( 150 ) NOT NULL ,
`title` VARCHAR( 100 ) NOT NULL ,
`author` VARCHAR( 75 ) NOT NULL
) ENGINE = INNODB;
```

This table will hold all the information we need about our favorite website.
Where to access it (the url field), what the title is, and who is the primary
author on the website.

This article is not designed to teach you SQL, so you may need to copy and paste
your way through for the time being. [W3 Schools has some SQL
tutorials][w3_schools] that you may find helpful.

### Connecting to the Database with PDO

PHP offers multiple interfaces for connecting and interacting with databases,
but I think we'll find [PHP Data Objects][pdo] the easiest to use.

```php
<?php
/*
 * global.php
 */
// tries to open a mysql connection
try {
  $dbh = new PDO('mysql:host=localhost;dbname=cmsTutorial', 'user', 'pass');
}catch( PDOException $e ) {
  echo $e->getMessage();
}
```

### Add Some Data

To make it easier for us begin working with our database we need some
information. The code below will go ahead an insert two rows into our table with
some websites that I like.

```php
<?php
/*
 * index.php
 */
include 'global.php';
/*********************
* Remove this line and below
* once you've loaded the page
*********************/
// Prepare our SQL query
$stmt = $dbh->prepare("INSERT INTO `websites` (`url`, `title`, `author`) VALUES (:url, :title, :author)");
// An array of items to insert
// notice how the keys reflect our query's [:url, :title, :author]
$websites = array(
  array(
    'url' => 'http://css-tricks.com',
    'title' => 'CSS-Tricks',
    'author' => 'Chris Coyier'
  ),
  array(
    'url' => 'http://net.tutsplus.com',
    'title' => 'NetTuts+',
    'author' => 'Jeffrey Way'
  )
);
// loop through each website and
// execute the query with the website data
foreach( $websites as $website ) {
  if( !$stmt->execute($website) ) {
    echo '<pre>', print_r($stmt->errorInfo(), true), '</pre>';
  }
}
```

### Displaying our Favorite Websites

Now that `index.php` has been cleared we can go ahead and add the code that we
need to make this website functional.

```php
<?php
/*
 * index.php
 */
include 'global.php';
// this will give us access to the Website class
include 'models/website.php';
// this time we'll run PDO::query because the SQL is static
$stmt = $dbh->query("SELECT `id`, `url`, `title`, `author` FROM `websites`");
// get all websites through our Website class
$websites = $stmt->fetchAll(PDO::FETCH_CLASS, 'Website');
?>
<!DOCTYPE html>
<html lang="en">
<head>
<title>My Favorite Website</title>

</head>
<body>

  <ol>
    <!-- loop through each website -->
    <?php foreach( $websites as $website ) : ?>
      <li>
        <!-- output: <a href="__URL__">__TITLE__</a> by: __AUTHOR__ -->
        <a href="<?php echo $website->url ?>"><?php echo $website->title ?></a>
        from: <?php echo $website->author ?>
      </li>
    <?php endforeach; ?>
  </ol>

  <a href="new.php">New Website</a>
</body>
</html>
```

This is all we need for our website class

```php
<?php
/*
 * models/website.php
 */
class Website {
  // for our own reference of what's available
  public $url,
         $title,
         $author;
  // allows us to easily set attributes
  // on the class
  function __construct(array $data = null) {
    if( empty($data) ) 
      return;
    foreach( $data as $field => $value ) {
      $this->$field = $value;
    }
  }
}
```

### Creating the Form

We are going to create a single file for the form because it will be used when
adding and editing website details.

```php
<form method="POST">

  <div class="field">
    <label for="url">URL:</label>
    <input type="text" for="url" name="website[url]" value="<?php echo $website->url ?>" />
  </div>

  <div class="field">
    <label for="title">Title:</label>
    <input type="text" for="title" name="website[title]" value="<?php echo $website->title ?>" />
  </div>

  <div class="field">
    <label for="author">Author:</label>
    <input type="text" for="author" name="website[author]" value="<?php echo $website->author ?>" />
  </div>

  <div class="field">
    <input type="submit" name="save_website" />
  </div>
</form>
```

There are two important things to note inside of our form.

1. Each input is set into a website[] array
2. The file requires us to have a $website variable

```php
<?php
/*
 * new.php
 */
include 'global.php';
include 'models/website.php';
// check if the form was submitted
if( isset($_POST['save_website']) ) {
  // create the website object
  $website = new Website($_POST['website']);
  // prepare an SQL query
  $stmt = $dbh->prepare("INSERT INTO `websites` (`url`, `title`, `author`) VALUES (:url, :title, :author)");
  // run the SQL query
  if( $stmt->execute(array(
        'url' => $website->url,
        'title' => $website->title,
        'author' => $website->author
      ))
  ) {
    // if successful then go back to home page
    header("Location: index.php");
  }else {
    // display an error if it failed
    echo "<p>failed to add website</p>";
  }
}else {
  // create the variabel so that we have access to it in our form
  $website = new Website;
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
<title>My Favorite Website</title>

</head>
<body>

  <?php include '_form.php' ?>

</body>
</html>
```

Notice how we check to see if a form has been submitted right at the beginning.
This let's us choose whether we want to try and insert a new row or display
the form.

Both sides of the `if` statement create the `$website` variable for the form.
This is cool because it will automatically fill in the inputs if the value has
been added to the object.

### Editing a Website

The first thing we need to do is add a link to `edit.php`

```php
<?php // this goes in index.php ?>
<li>
  <!-- output: <a href="__URL__">__TITLE</a> by: __AUTHOR__ -->
  <a href="<?php echo $website->url ?>"><?php echo $website->title ?></a>
  from: <?php echo $website->author ?> |
  <a href="edit.php?id=<?php echo $website->id ?>">Edit</a>
</li>
```

This will create a link to `edit.php?id=xxx` which we will be able to grab with
`$_GET['id']`.

```php
<?php
/*
 * edit.php
 */
include 'global.php';
include 'models/website.php';
// the id wasn't passed
if( !isset($_GET['id']) )
  header("Location: index.php");
// the form wasn't submitted
// then find the current website
if( !isset($_POST['save_website']) ) {
  // prepare the query to find the website by id
  $stmt = $dbh->prepare("SELECT `id`, `url`, `title`, `author` FROM `websites` WHERE `id` = :id");
  // execute the query with the current id
  $stmt->execute(array(
    'id' => $_GET['id']
  ));
  // set the website variable for the form
  $website = $stmt->fetchObject('Website');
}else { // the form was submitted
  // prepare the query to update the website's information
  $stmt = $dbh->prepare("UPDATE `websites` SET `url` = :url, `title` = :title, `author` = :author WHERE `id` = :id");
  // try to update the current website
  if( $stmt->execute($_POST['website'] + array('id' => $_GET['id'])) ) {
    header("Location: index.php");
  }else {
    echo "<p>Failed to update the website</p>";
  }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
<title>My Favorite Website</title>

</head>
<body>

  <?php include '_form.php' ?>

</body>
</html>
```

This requires a little more logic than when inserting a new record because we
have to find a current record and then update it.

Most of what I've done in this file is the same as the last few examples,
perhaps the most complex part is at

```php
if( $stmt->execute($_POST['website'] + array('id' => $_GET['id'])) ) {
```

basically it's concatenating the two arrays together resulting in

```php
array(
  'url' => '...',
  'title' => '...',
  'author' => '...',
  'id' => 123 // the new item
)
```

### Deleting Records

The first thing we need to do is add a link to `delete.php` in `index.php`

```php
<?php // this goes in index.php ?>
<!-- output: <a href="__URL__">__TITLE</a> by: __AUTHOR__ -->

from: <?php echo $website->author ?> |
<a href="edit.php?id=<?php echo $website->id ?>">Edit</a> |
<a href="delete.php?id=<?php echo $website->id ?>">Delete</a>
```

```php
<?php
/**
 * delete.php  
 */
 
include 'global.php';
// the id wasn't passed
if( !isset($_GET['id']) ) {
  header("Location: index.php");
} 
  
// prepare the query to find the website by id
$stmt = $dbh->prepare("DELETE FROM `websites` WHERE `id` = :id");
// execute the query with the current id
if( $stmt->execute(array('id' => $_GET['id'])) ) {
  header("Location: index.php");
}
echo "failed to delete the item";
```

There really isn't anything really special in this part. Once you've added a
page for inserting and updating, deleting is as easy is falling off a log.

### Conclusion

The hardest part of creating a CMS is learning the SQL, which means you'll have
to practice, practice, practice. Once you have a basic understanding of how to
write/run different queries based on what needs to be done you'll be good to go.

For practice I recommend creating a few basic websites to challenge yourself. A
blog that has posts, comments and categories would be a great start.

[w3_schools]: http://www.w3schools.com/sql/default.asp
[pdo]: http://php.net/pdo
