---
layout: post
title: Github Actions with Rails and Postgres
date: 2019-11-05 22:47:46 -0500
categories:
- Ruby on Rails
- Github
- Postgres
tags:
- ruby-on-rails
- github
- postgres
---

Github Actions were recently made widely available and this workflow has been
awesome. It runs Postgres in a docker container and installs the `libpq-dev`
package for the `pg` gem. Enjoy!

> It's important to note that this configuration runs Postgres inside a docker
> container with the port forwarded to a random, available port within the
> build's VM. Don't forget to modify your `config/database.yml` to read the
> configuration from the assigned environment variables.

```yaml
name: Ruby

on: [push]

jobs:
  rspec:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:10.8
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: RAILS_APP_DB_NAME_test
        ports:
        # will assign a random free host port
        - 5432/tcp
        # needed because the postgres container does not provide a healthcheck
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5

    steps:
    - uses: actions/checkout@v1

    - name: Install Postgres Libraries
      run: |
        sudo apt-get install libpq-dev

    - name: Set up Ruby 2.6
      uses: actions/setup-ruby@v1
      with:
        ruby-version: 2.6.x

    - name: Bundle Install
      run: |
        gem install bundler
        bundle install --jobs 4 --retry 3

    - name: Build and test with RSpec
      env:
        POSTGRES_HOST: localhost
        {% raw %}POSTGRES_PORT: ${{ job.services.postgres.ports[5432] }} # get randomly assigned published port{% endraw %}
        POSTGRES_USER: postgres
        POSTGRES_PASS: postgres
      run: |
        bundle exec rspec
```

I'm using `#fetch` to have sensible defaults if the environment variables aren't
defined.

```yml
test:
  <<: *default
  database: RAILS_APP_DB_NAME_test
  username: <%= ENV.fetch('POSTGRES_USER', '') %>
  password: <%= ENV.fetch('POSTGRES_PASS', '') %>
  host: <%= ENV.fetch('POSTGRES_HOST', 'localhost') %>
  port: <%= ENV.fetch('POSTGRES_PORT', 5432) %>
```
