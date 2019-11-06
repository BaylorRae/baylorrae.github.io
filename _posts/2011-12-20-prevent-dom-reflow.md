---
layout: post
title: Prevent DOM Reflow
date: 2011-12-20 14:29:43.000000000 -06:00
categories:
- Javascript
tags:
- createDocumentFragment
- how to
- javascript
- prevent
- reflow
meta:
  dsq_url: 'http://baylorrae.com/blog/2011/12/20/prevent-dom-reflow/'
  dsq_thread_id: '4 http://baylorrae.com/?p=4'
redirect_from: /blog/2011/12/20/prevent-dom-reflow/
---

Javascript is a really laid back language. You can write a script a hundred
different ways and most browsers don't care. However, as I found out there are
consequences for not diligently writing proper code.

I turned Chris Coyier's [datalist pollyfill] into a jQuery plugin. Everything
seemed perfect and I was rather self-impressed until I saw [this issue][issue]. I had
never heard of [DOM reflow] before and I was afraid it might be a difficult fix.
But after a few google searches it turned out to be really easy.

### What was wrong exactly

```js
var $stuff = $('#stuff');

$.each(('this is an array of things that will get added to $stuff').split(' '), function(i, item) {
  $('<li />', {
    text: item
  }).appendTo($stuff);
});
```

The problem generally occurs when you modify the DOM tree in a loop. In the code
above you can see that I'm appending 12 `<li>`s to my `<ol id="stuff">`. This
means that the browser is going to update the layout of the DOM 12 times in a
couple of seconds. That's a lot.

### The Fix

```js
var $stuff = $('#stuff'),
    temp_items = document.createDocumentFragment(),
    temp_item = null;

$.each(('this is an array of things that will get added to $stuff').split(' '), function(i, item) {
  temp_item = $('<li />', {
    text: item
  })[0];

  temp_items.appendChild(temp_item);
});

$stuff.append(temp_items);
```

The best way to fix the problem is to modify the DOM as few times as possible.
So I used [document.createDocumentFragment]. Why? Because the document fragment is
not part of the DOM tree. Instead it stays in memory and holds all the items you
append to it. Then when you're ready, you can append the `documentFragment` to
`$stuff`.

### Take it a step further

Using [jQuery.each] isn't as fast as a traditional for loop [see my test results].
You can run the same tests in your browser on [jsperf]. You may find it ironic
that I care because I used `.split()` instead of writing an array, but I'm fine
with it for this small example.

[datalist pollyfill]: https://github.com/chriscoyier/Relevant-Dropdowns
[issue]: https://github.com/chriscoyier/Relevant-Dropdowns/issues/5
[DOM reflow]: http://dev.opera.com/articles/view/efficient-javascript/?page=3#reflow
[document.createDocumentFragment]: https://developer.mozilla.org/en/DOM/document.createDocumentFragment
[jQuery.each]: http://api.jquery.com/jQuery.each/
[see my test results]: {{ site.baseurl }}/assets/Screen-Shot-2011-12-20-at-3.41.01-PM.png
[jsperf]: http://jsperf.com/jquery-each-vs-for-loop/43
