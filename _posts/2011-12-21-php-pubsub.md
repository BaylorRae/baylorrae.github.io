---
layout: post
title: PHP PubSub
date: 2011-12-21 08:00:29.000000000 -06:00
categories:
- PHP
- Snippets
- Tutorials
tags:
- example
- frameworks
- php
- pubsub
- tutorial
meta:
  dsq_thread_id: '835205007'
---

A little over a year ago I read a post on Nettuts+ regarding [PubSub with
jQuery][jquery_pubsub]. The idea is you create (or **sub**scribe to) a collection
of functions that are
called when you **pub**lish an event in your application.

It seems like something that only works in javascript because php doesn't deal
with user events. But even WordPress implements a PubSub like system with [action
and filter hooks][wordpress_actions].

### The Problem

On the surface PubSub looks a little redundant because you could use something
like this in your application.

```php
<?php
function before_save_record($record_data) {
    # code...
}
function after_save_record($record_data) {
    # code...
}
function save_record($record_data) {
    $record_data = before_save_record($record_data);
    # code ...
    after_save_record($record_data);
}
save_record(array(
    'username' => 'BaylorRae',
    'first_name' => 'Baylor',
    'last_name' => 'Rae\''
));
```

However, the above code isn't customizable. For instance, if this was packaged
into a framework I would have to change the core files anytime I needed to
change its functionality.

### The Solution

```php
<?php
/*
    Somewhere in the core files
*/
PubSub::subscribe('beforeSaveRecord', function($record_data) {
    # code...
});
PubSub::subscribe('afterSaveRecord', function($record_data) {
    # code...
});
function save_record($record_data) {
    $record_data = PubSub::publish('beforeSaveRecord', $record_data);
    # code ...
    PubSub::publish('after_save_record', $record_data);
}
// my code
PubSub::subscribe('beforeSaveRecord', function($record_data) {
    # do stuff with $record_data
});
save_record(array(
    'username' => 'BaylorRae',
    'first_name' => 'Baylor',
    'last_name' => 'Rae\''
));
```

Notice how I was able to extend the original subscription and do whatever I
wanted with `$record_data`.

### A realistic example

```php
<?php
// user_manager.php from xxx.com
PubSub::subscribe('saveLogin', function($user_token) {
    $_SESSION['user_token'] = $user_token;
});
// my_file.php
PubSub::subcribe('saveLogin', function($user_token) {
    Cookie::set('user_token', '1 year', $user_token);
});
```

Before you get all upset because I've assumed that everyone will adopt my code,
please understand this. This article is a concept/tutorial on how to build this
system. If you think your next project needs to implement PubSub you'll have
this article as a starting point and as a guide.

### Let's Roll! Adding Subscriptions

This is a very simple project. We're going to have a class that follows the
[singleton pattern]. This way we don't have to pass a variable around in the
application to keep up with all of the subscriptions.

```php
<?php
class PubSub {
  private static $events = array(); // all subscriptions
  // Don't allow PubSub to be initialized outside this class
  private function __construct() {}
  private function __clone() {}
  /**
   * Adds a subscription to the stack
   *
   * @param string $name 
   * @param function $callback needs to be callable with call_user_func
   * @return void
   * @author Baylor Rae'
   */
  public static function subscribe($name, $callback) {
    // Make sure the subscription isn't null
    if( empty(self::$events[$name]) )
      self::$events[$name] = array();
    // push the $callback onto the subscription stack
    array_push(self::$events[$name], $callback);
  }
}
// Basic usage
PubSub::subscribe('beforeSave', function() {
  echo 'PubSub::beforeSave';
});
PubSub::subscribe('afterSave', function() {
  echo 'PubSub::afterSave';
});
```

We set the `__construct()` and `__clone()` methods to private so they cannot be
called outside of the class. Even though we are not using them, it will break
the script if the class is initialized and used.

The meat of the class so far is in the `subscribe()` method. Basically we are
checking to see if the event has been subscribed to before. If it hasn't, then
we create an empty array so we can push callbacks to it.

The `$events` variable looks like this after the script has run.

```php
Array
(
    [beforeSave] => Array
        (
            [0] => Closure Object
                (
                )

        )

    [afterSave] => Array
        (
            [0] => Closure Object
                (
                )

        )

)
```

### Publishing our subscriptions

```php
<?php
class PubSub {
  private static $events = array(); // all subscriptions
  // Don't allow PubSub to be initialized outside this class
  private function __construct() {}
  private function __clone() {}
  /**
   * Adds a subscription to the stack
   *
   * @param string $name 
   * @param function $callback needs to be callable with call_user_func()
   * @return void
   * @author Baylor Rae'
   */
  public static function subscribe($name, $callback) {
    // Make sure the subscription isn't null
    if( empty(self::$events[$name]) )
      self::$events[$name] = array();
    // push the $callback onto the subscription stack
    array_push(self::$events[$name], $callback);
  }
  /**
   * Calls the last subscription in the stack
   *
   * @param string $name 
   * @param string $params 
   * @return false if fails
   * @author Baylor Rae'
   */
  public static function publish($name, $params = '') {
    // Check to see if the subscribe isn't null
    if( empty(self::$events[$name]) )
      return false;
    // Gets all parameters passed to the function
    // and removes the first, which is $name
    $params = func_get_args();
    array_shift($params);
    // If there's only one event, then call it and return the value
    if( count(self::$events[$name]) === 1 ) {
      if( is_callable(self::$events[$name][0]) )
        return call_user_func_array(self::$events[$name][0], $params);
      else
        return false;
    }
    // Loop through all the events and call them
    foreach( self::$events[$name] as $event ) {
      if( is_callable($event) )
        call_user_func_array($event, $params);
    }
  }
}
// Basic usage
PubSub::subscribe('beforeSave', function() {
  echo 'PubSub::beforeSave';
});
PubSub::subscribe('afterSave', function() {
  echo 'PubSub::afterSave';
});
PubSub::publish('beforeSave');
```

The `publish()` method doesn't do a lot more that `subscribe()`. It first checks
that the subscription has been set. Next it gets all the parameters passed
except the first. Then it checks to see if there is only one _callable_ event and
calls it. But if there are multiple events, then it loops and calls each one of
them.

### Unsubscribing

```php
<?php
class PubSub {
  private static $events = array(); // all subscriptions
  // Don't allow PubSub to be initialized outside this class
  private function __construct() {}
  private function __clone() {}
  /**
   * Adds a subscription to the stack
   *
   * @param string $name 
   * @param function $callback needs to be callable with call_user_func()
   * @return void
   * @author Baylor Rae'
   */
  public static function subscribe($name, $callback) {
    // Make sure the subscription isn't null
    if( empty(self::$events[$name]) )
      self::$events[$name] = array();
    // push the $callback onto the subscription stack
    array_push(self::$events[$name], $callback);
  }
  /**
   * Calls the last subscription in the stack
   *
   * @param string $name 
   * @param string $params 
   * @return false if fails
   * @author Baylor Rae'
   */
  public static function publish($name, $params = '') {
    // Check to see if the subscribe isn't null
    if( empty(self::$events[$name]) )
      return false;
    // Gets all parameters passed to the function
    // and removes the first, which is $name
    $params = func_get_args();
    array_shift($params);
    // If there's only one event, then call it and return the value
    if( count(self::$events[$name]) === 1 ) {
      if( is_callable(self::$events[$name][0]) )
        return call_user_func_array(self::$events[$name][0], $params);
      else
        return false;
    }
    // Loop through all the events and call them
    foreach( self::$events[$name] as $event ) {
      if( is_callable($event) )
        call_user_func_array($event, $params);
    }
  }
  public static function unsubscribe($name) {
    if( !empty(self::$events[$name]) )
      unset(self::$events[$name]);
  }
}
// Basic usage
PubSub::subscribe('beforeSave', function() {
  echo 'PubSub::beforeSave';
});
PubSub::subscribe('afterSave', function() {
  echo 'PubSub::afterSave';
});
PubSub::unsubscribe('beforeSave');
PubSub::publish('beforeSave');
```

This is dead simple. All we have to do is check if the subscription has been
created and remove it.

[View the final code on GitHub][gist]

[jquery_pubsub]: http://net.tutsplus.com/tutorials/javascript-ajax/loose-coupling-with-the-pubsub-plugin/
[wordpress_actions]: http://codex.wordpress.org/Plugin_API/Action_Reference
[singleton pattern]: http://php.net/manual/en/language.oop5.patterns.php#language.oop5.patterns.singleton
[gist]: https://gist.github.com/1504836
