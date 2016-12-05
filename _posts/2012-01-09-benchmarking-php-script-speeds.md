---
layout: post
title: Benchmarking PHP Script Speeds
date: 2012-01-09 08:00:12.000000000 -06:00
categories:
- PHP
- Projects
- Snippets
tags:
- benchmarking
- php
- script speed
meta:
  dsq_thread_id: '835245658'
---

I recently learned about [jsperf.com][jsperf] and I've really enjoyed looking at the speed
results of different pieces of code. However, I couldn't find any sites that did
the same for PHP and I've really wanted to test different ideas that I've had.

### How it should work

There were a couple of things that I wanted to make sure were done. The first
was to make it easy to define tests. I looked at [phpbench.com][phpbench] and liked how he
stored each test into a function. The second was that I wanted to create an loop
that would run for a specified amount of time.

### The syntax

I wanted a way to write write tests, give each one a name and set the allowed
time to run. If you've ever used [jsperf.com][jsperf] this should be very similar.

```php
<?php
require 'Benchmark.php';
// a place to store the result of each test
$results = array();
// my version of un-camel casing a word
function test_1() {
  return strtolower(implode(' ', preg_split('/(?<=\\w)([A-Z])/', 'aCamelCasedWord')));
}
// CakePHP's version of un-camel casing a word
function test_2() {
  return strtolower(preg_replace('/(?<=\\w)([A-Z])/', '_\\1', 'aCamelCasedWord'));
}
/**
 * @param title
 * @param function/test
 * @param allowed time to run
 */
$results[] = Benchmark::run_test('My Version', 'test_1', 10);
$results[] = Benchmark::run_test('Cake\'s Version', 'test_2', 10);
foreach( $results as $result )
  echo $result;
```

### The `Benchmark` class

```php
<?php
class Benchmark {
  /**
   * Creates a loop that lasts for $allowed_time and logs how many
   * times a function was able to run
   *
   * @param string $name the name of the test 
   * @param function $test the function to run
   * @param integer $allowed_time seconds to run a function
   * @return string
   * @author Baylor Rae'
   */
  public static function run_test($name, $test, $allowed_time = 10) {
    // get the time the function was called
    $start_time = microtime(true);
    // stores how many times $test was able to run
    $times_run = 0;
    // don't allow output
    ob_start();
    // run the $test function until time is up
    do {
      call_user_func($test);
      $times_run++;
    }while( number_format(microtime(true) - $start_time, 0) < $allowed_time);
    // end output buffering
    ob_end_clean();
    // return the formatted results
    return self::results($name, $times_run, $allowed_time);
  }
  /**
   * Formats results for easy reading
   *
   * @param string $name name of the test
   * @param integer $times_run number of times the test ran
   * @param integer $allowed_time how long the test was allowed to run
   * @return string
   * @author Baylor Rae'
   */
  private static function results($name, $times_run, $allowed_time) {
    $output = '<h2>Results for ' . $name . '</h2>';
    $output .= '<dl>';
      $output .= '<dt>Times Run</dt>';
      $output .= '<dd>' . number_format($times_run, 0) . '</dd>';
      $output .= '<dt>Ran For</dt>';
      $output .= '<dd>' . $allowed_time . 's</dd>';
    $output .= '</dl>';
    return $output;
  }
}
```

There isn't a whole lot to the code. When `Benchmark::run_test` is called it gets
the time it was called, turns on output buffering to prevent anything being
displayed on the page and freezing the browser, and then loops through the test
and counts the number of times it ran the function.

The `Benchmark::results`method formats the results to display the number of times
it ran and for how long it was allowed to run.

### Things to note

I haven't put this through a lot of tests. At this point I've really only
intended to use it to look at a couple of different concepts. I'm not sure how
it can handle a large amount of tests and wouldn't suggest doing a lot of tests
without adding a [sleep()][php_sleep] call between a couple of them.

I also wanted to create an online version that could act like
[jsperf.com][jsperf] for PHP. My biggest fear is the safety of my server and I'm
not sure what needs to be done to secure it. If anyone has an idea on what can
be done to prevent anything dangerous I'd really appreciate it.

[jsperf]: http://jsperf.com/
[phpbench]: http://www.phpbench.com/
[php_sleep]: http://php.net/sleep
