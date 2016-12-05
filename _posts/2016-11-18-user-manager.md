---
layout: post
title: Adding a User Manager to your Feature Tests
date: 2016-11-18 08:30:14.000000000 -06:00
categories:
- Acceptance Testing
tags:
- acceptance tests
- cucumber
- selenium
- tdd
meta:
  dsq_thread_id: '5313060455'
---

In my last post I described how to [write declarative step definitions][third_person_acceptance_tests].
However, this means our test cases can't assume it knows who is performing the
action because the user can change at any time. We'll need to introduce a sort
of state machine to track the current user and log in as another user when
needed.

I've written this example feature to purchase a product using a **buyer** and
**seller** with the **buyer** performing two steps in a row.

```gherkin
Feature: purchase product

  Scenario: purchase product
    Given seller has created a product
    When buyer purchases the product
    And buyer sends payment
    Then seller should receive money
    And the product shouldn't be listed
```

When writing our step definitions we will include a capture group to get the
username and then pass that to our `UserManager` class.

```ruby
Given(/^(.*) has created a product$/) do |username|
  UserManager.change_user(username)
end

When(/^(.*) purchases the product$/) do |username|
  UserManager.change_user(username)
end

When(/^(.*) sends payment$/) do |username|
  UserManager.change_user(username)
end

Then(/^(.*) should receive money$/) do |username|
  UserManager.change_user(username)
end

Then(/^the product shouldn't be listed$/) do
end
```

Now for the final piece, the `UserManager` class. This is a very simple class
with a single method to update the `current_username` variable if it is
different. When this variable changes it will tell our `LoginPage` to log us in
with the new username.

```ruby
class LoginPage
  def self.login_as(username)
    # fill in login fields and submit form here
    puts "Logged in as #{username}"
  end
end

class UserManager
  def self.change_user(username)
    return if @current_username == username
    @current_username = username
    LoginPage.login_as(username)
  end
end
```

As you can see this is a very small change to make and allows our step
definitions to simply carry out the instructions from the high level scenario.

[third_person_acceptance_tests]: {% post_url 2016-11-08-writing-acceptance-tests-in-third-person %}
