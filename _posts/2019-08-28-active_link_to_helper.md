---
layout: post
title: Active Link To Helper
date: 2019-08-28 10:19:46 -0500
categories:
- Ruby on Rails
tags:
- ruby-on-rails
- snippets
- link_to
---

I'm often looking for a flexible solution for determining if I'm on the current
route. I say flexible because sometimes a link is bound to a single controller,
or sometimes multiple controllers, or multiple controllers with specific
actions.

<ul>
  <li><a href="#helper">Rails Helper</a></li>
  <li><a href="#matcher">Matcher Class</a></li>
  <li><a href="#usage">Usage</a></li>
  <li><a href="#spec">Spec</a></li>
</ul>

<h3 id="helper">Rails Helper</h3>

```ruby
# app/helpers/active_link_helper.rb
# frozen_string_literal: true

require_relative "../../lib/active_link_path"

module ActiveLinkHelper
  def nav_link_to(path, matches: [], &block)
    matcher = ActiveLinkPath.new(params)
    active = matcher.active_match?(matches) ? 'active' : nil
    link_to path, class: ['nav-link', active].compact, &block
  end
end
```

<h3 id="matcher">Matcher Class</h3>

```ruby
# lib/active_link_path.rb

class ActiveLinkPath
  attr_reader :context

  def initialize(context)
    @context = context.stringify_keys
  end

  def active_match?(match)
    matches = match.is_a?(Hash) ? [match] : match

    matches.any? do |match|
      match.stringify_keys!
      values_match(match, context.slice(*match.keys))
    end
  end

  private

  def values_match(match, sliced_context)
    match.all? do |(key, value)|
      value = Array(value)
      value.include?(sliced_context[key])
    end
  end
end
```

<h3 id="usage">Usage</h3>

The API is somewhat complex to accommodate most use cases.

1. Basic resource `resources :products`
2. Nested resources `resources :collections { resources :products }`
3. Top level REST actions `get '/export/products', to: 'export#products'`

In the above list, #3 is the most interesting since you'd probably access this
action from the products index page (`/products`).

```ruby
nav_link_to(some_path,
            matches: {controller: 'products'})

# 👍   /products                 products#index
# 👍   /products/1               products#show
# 👍   /products/1/edit          products#edit
# 👍   /collections/1/products   products#index
# ❌   /collections              collections#index
# ❌   /collections/1            collections#show
# ❌   /export/products          export#products
```

<hr />

```ruby
nav_link_to(some_path,
            matches: {controller: %w(products collections)})

# 👍   /products                 products#index
# 👍   /products/1               products#show
# 👍   /products/1/edit          products#edit
# 👍   /collections/1/products   products#index
# 👍   /collections              collections#index
# 👍   /collections/1            collections#show
# ❌   /export/products          export#products
```

<hr />

```ruby
nav_link_to(some_path,
            matches: {controller: 'products',
                      action: 'show'})

# 👍   /products/1               products#show
# ❌   /products                 products#index
# ❌   /products/1/edit          products#edit
# ❌   /collections/1/products   products#index
# ❌   /collections              collections#index
# ❌   /collections/1            collections#show
# ❌   /export/products          export#products
```

<hr />

```ruby
nav_link_to(some_path,
            matches: {controller: %w(products collections),
                      action: 'show'})

# 👍   /products/1               products#show
# 👍   /collections/1            collections#show
# ❌   /products                 products#index
# ❌   /products/1/edit          products#edit
# ❌   /collections/1/products   products#index
# ❌   /collections              collections#index
# ❌   /export/products          export#products
```

<hr />

```ruby
nav_link_to(some_path,
            matches: [
              {controller: %w(products collections), action: 'index'},
              {controller: 'products', action: 'edit'},
              {controller: 'export', action: 'products'},
             ])

# 👍   /products                 products#index
# 👍   /products/1/edit          products#edit
# 👍   /collections/1/products   products#index
# 👍   /collections              collections#index
# 👍   /export/products          export#products
# ❌   /products/1               products#show
# ❌   /collections/1            collections#show
```

<hr />

<h3 id="spec">Spec</h3>

I've also written a spec for this if you'd like to include it in your
application.

```ruby
# spec/lib/active_link_path_spec.rb
# frozen_string_literal: true

require "spec_helper"
require "active_support/hash_with_indifferent_access"
require "./lib/active_link_path"

describe ActiveLinkPath do
  context "active_match?" do
    it "finds overlappings key value pairs" do
      matcher = ActiveLinkPath.new({'key' => 'value1', 'key2' => 'value2'})
      expect(matcher.active_match?(key: 'value1')).to be_truthy
      expect(matcher.active_match?(key: 'value2')).to be_falsey
    end

    it "finds from multiple matching pairs" do
      matcher = ActiveLinkPath.new({'key' => 'value1', 'key2' => 'value2'})
      expect(matcher.active_match?(key: 'value1', key2: 'value2')).to be_truthy
      expect(matcher.active_match?(key: 'value1', key2: 'value1')).to be_falsey
      expect(matcher.active_match?(key: 'value1', key3: 'value3')).to be_falsey
    end

    it "allows an array of values" do
      matcher = ActiveLinkPath.new({'key' => 'value1', 'key2' => 'value2'})
      expect(matcher.active_match?(key: ['value', 'value1'])).to be_truthy
      expect(matcher.active_match?(key: ['value', 'value2'])).to be_falsey
    end

    it "allows an array of matchers" do
      matcher = ActiveLinkPath.new({'key' => 'value1', 'key2' => 'value2'})
      expect(
        matcher.active_match?([
          {key: ['value'], key2: 'value3'},
          {key: 'value1', key2: ['value', 'value2']}
        ])
      ).to be_truthy

      expect(
        matcher.active_match?([
          {key: ['value'], key2: 'value3'},
          {key: 'value1', key2: ['value', 'value1']}
        ])
      ).to be_falsey
    end
  end
end


```
