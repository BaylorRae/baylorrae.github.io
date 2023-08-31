---
layout: post
title: Wrangle Your SQL With Arel by Eric Hayes
date: 2020-09-26 08:00:00 -0500
categories:
- ActiveRecord
- RSpec
tags:
- activerecord
- rspec
---

This is a fascinating talk that walks through the underlying APIs used to build SQL queries in ActiveRecord. Eric does a great job showing how ORMs use Abstract Syntax Trees to create SQL queries from hashes and method calls.

The end result shows how they built an incredibly complex cohort query with efficiency and maintainablity using Arel.

<div class="embed-container">
  <iframe
      src="https://www.youtube.com/embed/8pQGzsDzYEo"
      width="700"
      height="480"
      frameborder="0"
      allowfullscreen="true">
  </iframe>
</div>