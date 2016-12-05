---
layout: post
title: Writing Acceptance Tests in Third Person
date: 2016-11-08 10:13:53.000000000 -06:00
categories:
- Acceptance Testing
tags:
- acceptance tests
- cucumber
- selenium
- tdd
meta:
  dsq_thread_id: '5288684885'
redirect_from: /blog/2016/11/08/writing-acceptance-tests-in-third-person/
---

When I was learning to write acceptance tests with Cucumber everything started
with "I". It made sense because you were driving a browser to click buttons and
fill in forms and that's what "you" did. However, I recently realized this
doesn't work when writing tests that require multiple users to interact with the
same test. Instead acceptance tests should be written in third person as
instructions of what should be done.

Let's say we are building a trading site where two users can offer a price for
an item that can be accepted or rejected. If we wrote this in first person our
cucumber feature might look like the following.

```gherkin
Feature: offer price
  Scenario: accept offer price
    Given I offer a price of "5.75"
    When the price is accepted
    Then I should be able to send payment
```

When reading this test we have to assume the test will be run as the buyer in
this case because we are offering a price for a product and then sending the
payment. We also have to assume the action is being done by the seller outside
this test as it would be unexpected for the test to log in as the seller to
accept the offer price.

If we write this test in third person it becomes much clearer as to who is
performing which action in the test.

```gherkin
Feature: offer price
  Scenario: accept offer price
    Given buyer offers a price of "5.75"
    When seller accepts the price
    Then buyer should be able to send payment
```

This test immediately explains the buyer is offering a price, the seller accepts
and the buyer then sends payment. In addition, when running the test it is
expected that the seller will log into the site and accept the price rather than
being a surprise.

Writing tests in third person also gives us the ability to include unlimited
actors in the test without having to do weird tricks to switch accounts.

```gherkin
Feature: offer price
  Scenario: accept offer price
    Given buyer offers a price of "5.75"
    And buyer2 offers a price of "6.00"
    When seller accepts the "6.00" price
    Then buyer should not be able to send payment
    And buyer2 should be able to send payment
```
