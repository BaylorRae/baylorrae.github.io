---
layout: post
title: Creating a cycle() function in PHP
date: 2012-01-24 13:56:11.000000000 -06:00
categories:
- PHP
- Snippets
tags:
- cycle function
- php
- rails
- snippet
meta:
  dsq_thread_id: '246 http://baylorrae.com/?p=246'
redirect_from: /blog/2012/01/24/creating-a-cycle-function-in-php/
---

I love using the `cycle` helper in Rails and I've always wished PHP had something
similar. But every time I created one I ran into the same problem every time.
How do I use multiple cycle calls in a single loop?

I happened to look at [Rails' source code][rails_source] and noticed it allowed a specific
namespace to be set in the last parameter `cycle('blue', 'red', name: 'colors')`.
This concept could be easily recreated with PHP by beginning the last argument
with a colon.

### The Function

```php
<?php
/**
 * Cycles through each argument added
 * Based on Rails `cycle` method
 * 
 * if last argument begins with ":" 
 * then it will change the namespace
 * (allows multiple cycle calls within a loop)
 * 
 * @param mixed $values infinite amount can be added
 * @return mixed
 * @author Baylor Rae'
 */
function cycle($first_value, $values = '*') {
  // keeps up with all counters
  static $count = array();
  // get all arguments passed
  $values = func_get_args();
  // set the default name to use
  $name = 'default';
  // check if last items begins with ":"
  $last_item = end($values);
  if( substr($last_item, 0, 1) === ':' ) {
    // change the name to the new one
    $name = substr($last_item, 1);
    // remove the last item from the array
    array_pop($values);
  }
  // make sure counter starts at zero for the name
  if( !isset($count[$name]) )
    $count[$name] = 0;
  // get the current item for its index
  $index = $count[$name] % count($values);
  // update the count and return the value
  $count[$name]++;
  return $values[$index];  
}
```

### Basic Usage

```php
<?php for( $i = 0; $i < 9; $i++ ) : ?>
  <p><?php echo cycle('one', 'two', 'three') ?></p>
<?php endfor; ?>
```

### Custom Namespace

```php
<?php for( $i = 0; $i < 9; $i++ ) : ?>
  <p class="<?php echo cycle('odd', 'even') ?>">
    <?php echo cycle('one', 'two', 'three', ':counter') ?>
  </p>
<?php endfor; ?>
```

### A Realistic Example

```php
<?php
  $things = array('shoe', 'sox', 'sand', 'laptop', 'guitar', 'bag', 'legos', 'elephpant', 'stapler', 'binder', 'desk', 'chair');
  foreach( $things as $thing ) :
?>
  <div class="grid_4 <?php echo cycle('alpha', '', 'omega') ?>">
    <?php echo $thing ?>
  </div>

  <?php if( cycle(false, false, true, ':use_clear_div') ) : ?>
    <div class="clear">// clear div every three items</div>
  <?php endif; ?>
<?php endforeach; ?>
```

This is a example of using `cycle` with the [960 Grid System][960gs]. The idea is instead
of keeping up with an iterating variable (probably `$i`), you simply pass a
matching number of boolean arguments to `cycle`.

[rails_source]: https://github.com/rails/rails/blob/196407c54f0736c275d2ad4e6f8b0ac55360ad95/actionpack/lib/action_view/helpers/text_helper.rb#L308
[960gs]: http://960.gs
