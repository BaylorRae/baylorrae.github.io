---
layout: post
title: Lazy Loading with PHP
date: 2011-12-23 08:00:34.000000000 -06:00
categories:
- PHP
tags: []
meta:
  dsq_thread_id: '147 http://baylorrae.com/?p=147'
redirect_from: /blog/2011/12/23/lazy-loading-with-php/
---

I've been looking through a lot of PHP projects on GitHub lately and I keep
seeing people including all their classes on launch. And unfortunately it's the
reason I decide not to use their code.

### What is Lazy Loading

Lazy loading it a pattern of only including classes when they are needed. And
fortunately enough PHP makes it super easy to implement.

I realize that I could have used require 'classes/user.php';. But as the
application grows and I have more classes I don't have to worry about including
each file. I used this same pattern in my [Huck PHP Testing Framework][huck_php].

In my framework my autoload setup worked like this though.

```php
<?php
// Allows multiple __autoload stacks
// and keeps autoloader in Huck class
spl_autoload_register(array('Huck', 'autoload'));
class Huck {
    public static function autoload($class_name) {
        if( substr($class_name, 0, 5) === 'Huck_' ) {
            $file = dirname(__FILE__) . '/' . substr($class_name, 5) . '.php';
            if( file_exists($file) )
                require_once $file;
        }
    }
}
```

This is my preferred way of autoloading because it allows multiple autoloaders
to be registered. This way if someone includes `Huck` in their application that
already has an `__autoload` in place, mine doesn't override it.

The [substr()][php_substr] function is called to make sure the `$class_name`
starts with `Huck_`. The second time I call it I remove `Huck_` so I get
whatever is left.

[huck_php]: {% post_url 2011-12-22-huck-php-testing-framework %}
[php_substr]: http://php.net/substr

