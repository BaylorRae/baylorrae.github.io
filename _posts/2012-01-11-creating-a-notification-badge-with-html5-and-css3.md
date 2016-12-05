---
layout: post
title: Creating a Notification Badge with HTML5 and CSS3
date: 2012-01-11 13:09:33.000000000 -06:00
categories:
- CSS3
- HTML5
- Tutorials
tags:
- css attr function
- css3
- data- attribute
- html5
- notification badge
meta:
  dsq_thread_id: '832225713'
redirect_from: /blog/2012/01/11/creating-a-notification-badge-with-html5-and-css3/
---

I recently wanted to add a notification badge to a website. I wasn't sure what
the best approach was, but I knew I didn't want to add extra markup and position
it with CSS. I also didn't want to use a jQuery plugin to add this little
feature.

I ultimately decided to use the HTML5 [data-][data_attribute] attribute with the
CSS [attr()][css_attr] function. It works in most browsers including IE 8+ which
was good enough for me because it wasn't a required feature on the site.

### The HTML

```html
<ol>
  <li>
    <a data-notifications="10" class="button" href="#">New Comments on Your Posts</a>
  </li>
  <li>
    <a class="button" href="#">Number of Post Likes :)</a>
  </li>
<ol>
```

All the magic is in the `data-notifications` attribute. It's really easy to add
to any element and can be easily grabbed with CSS.

### The CSS

```css
/* make sure the element has position: relative */
[data-notifications] {
  position: relative;
}

/* append the notification badge after it */
[data-notifications]:after {

  /* the burger */
  content: attr(data-notifications);

  /* the fries */
  position: absolute;
  background: red;
  border-radius: 50%;
  display: inline-block;
  padding: 0.3em;
  color: #f2f2f2;
  right: -15px;
  top: -15px;   
}
```

This is a great example of why I love the simplicity of CSS. The seemingly
complex idea of appending the `data-notifications` value is actually really
simple. Most of the CSS is just making it look good.

### The Demo

<pre class="codepen" data-height="300" data-type="result" data-href="eCFId" data-user="BaylorRae" data-safe="true"><code></code><a href="http://codepen.io/BaylorRae/pen/eCFId">Check out this Pen!</a></pre>

<script async src="http://codepen.io/assets/embed/ei.js"></script>

[data_attribute]: http://www.whatwg.org/specs/web-apps/current-work/multipage/elements.html#custom-data-attribute
[css_attr]: https://developer.mozilla.org/en/CSS/attr
