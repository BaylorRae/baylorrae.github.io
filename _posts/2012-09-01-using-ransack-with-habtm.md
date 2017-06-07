---
layout: post
title: Using Ransack with HABTM
date: 2012-09-01 08:00:00.000000000 -05:00
comments: true
categories:
- Filter Records
- has_many :through
- Rails
tags:
- filter records
- has_many through
- rails
- ransack
meta:
  dsq_thread_id: '832222202'
redirect_from: /blog/2012/09/01/using-ransack-with-habtm/
---

If you're unfamiliar with [Ransack][ransack], it's basically an advanced search form
generator for Rails. It allows you to add fields to the form for things like
`category_id_eq` and `title_cont` and it will "magically" query the database.

The only real problem with Ransack is the lack of documentation. Almost every
time I had a problem I ended up looking on [Stack Overflow][ransack_questions]
or through the `Github Issues` page. Both of those sites are easy to search
through which wasn't a hindrance, but it took a long time to figure out simple
things.

One of those simple things, which [I'm not the only one][stackoverflow_question] that had a problem with,
was creating a select field for a `has_many :through` or `has_and_belongs_to_many`
association.

### The Setup

Here are the three models that I have in place. It's a basic setup for `has_many
:through` allowing multiple categories being assigned to products and vice
versa. For more information take a look at [Rails Guide on ActiveRecord
associations][rails_guide].

```ruby
# product.rb
class Product < ActiveRecord::Base
  has_many :categorizations
  has_many :categories, :through => :categorizations
end

# category.rb
class Category < ActiveRecord::Base
  has_many :categorizations
  has_many :products, :through => :categorizations
end

# categorization.rb
class Categorization < ActiveRecord::Base
  belongs_to :category
  belongs_to :product
end
```

### The Ransack Way

For Ransack to pick up on the association you have to provide the "associated"
models plural name. For instance, if we are building a search form for the
products page then the field name would be
<code><strong>categories</strong>_id_eq</code>. Here's an example form

```erb
<%= search_form_for @query do |f| %>
  <%= f.label :categories_id_eq, "Category" %>
  <%= f.collection_select :categories_id_eq, Category.order(:title), :id, :title %>

  <%= f.submit "Filter Products" %>
<% end %>
```

[ransack]: https://github.com/ernie/ransack
[ransack_questions]: http://stackoverflow.com/questions/tagged/ransack
[github_issues]: https://github.com/ernie/ransack/issues
[stackoverflow_question]: http://stackoverflow.com/q/11619246/467546
[rails_guide]: http://guides.rubyonrails.org/association_basics.html#the-has_many-through-association
