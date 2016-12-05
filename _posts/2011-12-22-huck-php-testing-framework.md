---
layout: post
title: Huck - PHP Testing Framework
date: 2011-12-22 00:45:50.000000000 -06:00
meta:
  dsq_thread_id: '834069655'
---

```php
<?php
describe('Huck Fin', function() {

   it('should be Huck', function() {
      expect('Huck')->toBe('Huck'); 
   });

   it('should not be Huck', function() {
      expect('Tom')->not->toBe('Huck'); 
   });

});
```

Huck is a Behavior Driven Development (BDD) framework based on
[Jasmine][jasmine] for JS. It is designed with a simple syntax to give it a
fast learning curve.

### Getting Started

Huck runs in the browser so you don't have to load terminal. Just upload it to
your server, create the specs and load `index.php`. Huck will automatically find
all files in `specs/*_spec.php` and run them.

### Why did I write this?

I know there are a bunch of test suites for PHP, but I wanted to build my own
for the experience. Plus, I didn't like the syntax or result format other suites
had.

> Whenever you are tempted to type something into a print statement or a debugger expression, write it as a test instead.
> Martin Fowler

### Writing a Spec

```php
<?php
// specs/first_spec.php
describe('This is my Test Suite', function() {

});
```

All specs need to be placed inside of the `./specs` directory and the filename
must end in `_spec.php` for Huck to find it. Huck will autoload each file and run
the tests so you don't have to worry about that part.

```php
<?php
require '/path/to/User.php';

// specs/user_spec.php
describe('User Class Functionality', function() {

    it('should authenticate user', function() {
       expect(User::authenticate('BaylorRae', 'pass'))->toBeInstanceOf('User'); 
    });

    it('should not authenticate user', function() {
       expect(User::authenticate('BaylorRae', 'wrong password'))->not->toBeInstanceOf('User'); 
    });

});
```

Once you have described your spec you can add your tests. The syntax is designed
so you can read each line and understand what's going on.

Notice that in the second test I was able to make sure that it did **not** return an
instance of User. Normally the `User::authenticate()` method would return false
and the test would be `expect($x)->toBeFalsy()` but I wanted to show how to
invert the test.

### Matchers and how to write them

Huck comes with a number of matchers that should be more than enough in most
cases. The full list is on [GitHub in the Readme][huck_matchers].

#### Here are a couple of my favorites

```php
<?php
expect($x)->toMatch($pattern) # compares $x to regular expression $pattern
expect($x)->toBeArray() # makes sure $x is an array
expect($x)->toBeTruthy() # checks to see if $x === true
```

### The Syntax

```php
<?php

/**
 * Checks to match length of an array
 *
 * @param $actual the value passed into expect()
 * @param $expected the value passed into ->toBeLength()
 * @author Baylor Rae'
 */
Huck::addMatcher('toBeLength', function($actual, $expected) {
    if( !is_array($actual) && !is_array($expected) )
        return false;

    return count($actual) === count($expected);
});
```

Let me know what you think and/or what you would like to see added to framework.
This is the first testing framework that I've written and I'm not completely
sure everything is perfect yet. If you have any suggestions please let me know.

[View the project on GitHub][huck_php]

[jasmine]: https://github.com/pivotal/jasmine
[huck_matches]: https://github.com/BaylorRae/PHP-Huck#readme
[huck_php]: https://github.com/BaylorRae/PHP-Huck
