---
layout: post
title: Testing APIs
date: 2017-01-24 21:25:00.000000000 -06:00
categories:
- Acceptance Testing
tags:
- acceptance tests
- cucumber
- tdd
- api
---

I recently had the opportunity to test an API and wanted to validate the
results along with the structure of the response. When looking around on the
internet for inspiration on how to solve this I was intrigued by the wonderful
DSL created by [Frisby].

My goal was to not only test the security and correctness internally but also
the "user interface" of response codes and structure. I think the later is
extremely important due to the nature of objects being easily changeable
without any restrictions by the framework.

Frisby's DSL solves this by having chainable method calls to validate the `http
status`, `json schema` and `json body`. All three of which are the "user
interface" our customers would be looking at on a daily basis.

The following class provides a similar DSL for validating requests sent with `Rack::Test`. The JSON schema is validated with the [json-schema] gem.

```ruby
class ValidateResponse
  include RSpec::Matchers

  attr_reader :response

  delegate :status_code, to: Rack::Utils

  def initialize(response)
    @response = response
  end

  def self.with(response)
    new(response)
  end

  def expect_status(status)
    expect(status_code(response.status)).to eq(status_code(status))
    self
  end

  def expect_schema(schema)
    errors = JSON::Validator.fully_validate(schema, response.body)

    if errors.any?
      raise RSpec::Expectations::ExpectationNotMetError.new(errors)
    end

    self
  end

  def expect_body(body)
    expect(JSON.parse(response.body)).to eq(body)
    self
  end

end
```

Using this class is very simple and combined with the [`last_response`] method provided by `Rack::Test` makes this a breeze.

```ruby
ValidateResponse.with(last_response)
  .expect_status(:ok)
  .expect_schema("./features/api/schemas/projects/index.json")
  .expect_body([
    {
      "id" => 1,
      "title" => "project-title",
      "created_at" => "2017-01-01T00:00:00.000Z",
      "updated_at" => "2017-01-01T00:00:00.000Z"
    }
  ])
```

With the `ValidateResponse` class in place validating the important public facing parts of our API can be done quickly with a pleasant DSL.

[Frisby]: https://github.com/vlucas/frisby#creating-tests
[json-schema]: https://github.com/ruby-json-schema/json-schema
[`last_response`]: http://www.rubydoc.info/github/brynary/rack-test/Rack/MockSession:last_response
