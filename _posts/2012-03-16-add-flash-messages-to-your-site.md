---
layout: post
title: Add Flash Messages to Your Site
date: 2012-03-16 08:00:22.000000000 -05:00
categories:
- PHP
- Tutorials
tags:
- php
- tutorial
- user experience
- ux
meta:
  dsq_thread_id: '832224789'
redirect_from: /blog/2012/03/16/add-flash-messages-to-your-site/
---

Flash messages are often used to give feedback to the user when an action has
taken place. Such as, when they have logged in or tried to access a page without
permission.

### How it Works

Flash messages are set in `$_SESSION['flash_messages']`. At the
beginning of every page request we store them in a variable and reset the
`$_SESSION`. This gives us a chance to display the flash message, and prevents
us from accidentally displaying it twice.

### The Code

```php
<?php
// make sure sessions work on the page
session_start();

class Flash {

  // where all messages are stored
  public static $messages = array();

  /*
   * A generic function to store flash messages
   *
   * Flash::add('notice', 'a message to display');
   *
   * @param string $name the name/id of the flash
   * @param string $message the message to display
   */
  public static function add($name, $message) {
    $_SESSION['flash_messages'][$name] = $message;
  }

  /*
   * A shortcut to Flash::add()
   *
   * Flash::notice('a message to display');
   */
  public static function __callStatic($fn, $args) {
    call_user_func_array(array('Flash', 'add'), array($fn, $args[0]));
  }
}

// if $_SESSION['flash_messages'] isset
// then save them to our class
if( isset($_SESSION['flash_messages']) ) {
  self::$messages = $_SESSION['flash_messages'];
}

// reset the session's value
$_SESSION['flash_messages'] = array();
```

The class's job is to provide methods for us to add new messages. The work is
done at the end where we store and reset the flash messages.

### How to use the class

```php
<?php

include 'flash.php';

// using Flash::add directly
Flash::add('notice', 'Awesome job dude!');

// using the alias
Flash::alert('lol, you can #39;t do that!');
// calls: Flash::add('alert', 'lol, you can #39;t do that!');
```

### Displaying flash messages

```php
<?php include 'flash.php'; ?>
...
<?php foreach( Flash::$messages as $id => $msg ) : ?>
  <div class="flash_<?php echo $id ?>"><?php echo $msg ?></div>
<?php endforeach; ?>
```

As you can see it is very easy to display the messages and you can have them
implemented in no time at all.
