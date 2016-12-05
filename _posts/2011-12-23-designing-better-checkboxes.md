---
layout: post
title: Designing better Checkboxes
date: 2011-12-23 13:41:05.000000000 -06:00
categories:
- CSS3
- HTML5
tags:
- css3
- example
- user experience
- ux
meta:
  dsq_thread_id: '832827350'
---

When adding an `<input type="(radio|checkbox)" />` I recommend wrapping the whole thing in a label. Because it makes the clickable area bigger and allows the user to miss the text and input while getting the desired result.

Here's the basic markup.

```html
<label>
  <input type="checkbox">
  Is this awesome?
</label>
```

<pre class="codepen" data-height="300" data-type="result" data-href="yVNZwe" data-user="BaylorRae" data-safe="true"><code></code><a href="http://codepen.io/BaylorRae/pen/eCFId">Check out this Pen!</a></pre>

<script async src="http://codepen.io/assets/embed/ei.js"></script>


