---
layout: post
title: Rails `link_to_with_notification` Helper Method
date: 2012-01-16 11:03:34.000000000 -06:00
categories:
- HTML5
- Rails
- Tutorials
tags:
- custom helper method
- notification badge
- rails
- tutorial
meta:
  dsq_thread_id: '834967715'
---

In with my last post on [creating a notification badge][notification_badge] I showed the HTML and CSS
markup. But I forgot that even when the notification count was at zero it still
showed the badge.

### The Problem

Because was using a Rails app I wanted to continue using the `link_to` helper method.

### The Solution

I created a new helper method called `link_to_with_notifications` that allowed me
to use all the features of `link_to` and only add the badge if the value was not
zero.

### The Code

```ruby
def link_to_with_notifications(*args, &block)

  # get notification count from hash
  notifications = args[2][:notifications] || 0

  # create the data- attribute unless notifications.zero?
  args[2]['data-notifications'] = notifications unless notifications.zero?

  # delete original notifications hash
  args[2].delete(:notifications)

  # run original link_to helper
  link_to(*args, &block)
end
```

### Usage

```ruby
<%= link_to_with_notifications 'Completed Orders',
  user_carts_path(user, view: 'completed'),
  class: 'button',
  notifications: user.shopping_carts.where('completed = ?', true).count %>
```

[notification_badge]: {% post_url 2012-01-11-creating-a-notification-badge-with-html5-and-css3 %}
