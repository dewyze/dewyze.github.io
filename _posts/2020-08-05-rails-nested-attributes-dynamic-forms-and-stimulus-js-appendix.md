---
layout: post
title:  "Rails Nested Attributes, Dynamic Forms, and StimulusJS Appendix"
date:   2020-08-05
permalink: "/blog/rails-nested-attributes-dynamic-forms-and-stimulus-js-appendix"
tags: rails stimulus frontend
exclude_from_feed: true
---

1. [Rails Nested Attributes, Dynamic Forms, and StimulusJS Part 1](/blog/rails-nested-attributes-dynamic-forms-and-stimulus-js-part-1)
1. [Rails Nested Attributes, Dynamic Forms, and StimulusJS Part 2](/blog/rails-nested-attributes-dynamic-forms-and-stimulus-js-part-2)
1. Rails Nested Attributes, Dynamic Forms, and StimulusJS Appendix

If you want to follow along with the same controllers, models, views, and
routes, you can use these snippets. It does assume you have a working rails app.

## Models

### Store

```ruby
# app/models/store.rb

class Store < ApplicationRecord
  has_many :books, dependent: :destroy

  accepts_nested_attributes_for :books, allow_destroy: true, reject_if: :reject_books

  def reject_books(attributes)
    attributes["title"].blank?
  end
end
```

```sh
bin/rails generate migration CreateStores name:string
```

### Book

```ruby
# app/models/book.rb

class Book < ApplicationRecord
  belongs_to :store
end
```

```sh
bin/rails generate migration CreateBooks title:string store:belongs_to
```

Set up your database: `bin/rails db:migrate`.

Finally, let's set up our controller. Let's focus on the edit/update action and
assume we have an existing store.

## Controller & Route

```ruby
# app/controllers/stores_controller.rb

class StoresController < ApplicationController
  before_action :find_store

  def show; end

  def edit; end

  def update
    if @store.update(update_params)
      redirect_to @store
    else
      render :edit
    end
  end

  private

  def find_store
    @store = Store.includes(:books).find(params[:id])
  end

  def update_params
    params.require(:store).permit(:id, :name, books_attributes: %i[id title _destroy])
  end
end
```

```ruby
# config/routes.rb

Rails.application.routes.draw do
  resources :stores, only: %i[show edit update]
end
```

## DB Records

Open up a rails console with `bin/rails console` and create a Store:

```ruby
store = Store.new(name: "Waldenbooks")
store.books.build(title: "The Lord of the Rings")
store.save
```

## Views

My default body layout looks like this:

```erb
<body>
  <div class="container">
    <div class="box">
      <%= yield %>
    </div>
  </div>
</body>
```

And we'll add a show page:

```erb
<%# app/views/stores/show.html.erb %>
<h1><%= @store.name %></h1>
<h2>Books</h2>
<% @store.books.each do |book| %>
  <p><%= book.title %></p>
<% end %>
<%= link_to "Edit", edit_store_path(@store) %>
```

And the styling looks like this:

```scss
html,
body,
label,
input,
a,
button {
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  display: block;
  font-size: 16px;
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

h1,
h2,
h3,
h4,
h5 {
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  display: block;
  box-sizing: border-box;
}

label {
  margin-top: 16px;
}

input {
  margin-top: 8px;
  box-shadow: inset 0 0.0625em 0.125em rgba(10, 10, 10, 0.05);
  max-width: 100%;
  width: 100%;
  background-color: white;
  border-radius: 4px;
  color: #363636;
  -webkit-appearance: none;
  -moz-appearance: none;
  align-items: center;
  border: 1px solid #dbdbdb;
  height: 2.5em;
  line-height: 1.5;
  padding: 0px 10px;
  flex: 1 0 0;
}

.fields {
  display: flex;
}

button {
  cursor: pointer;
  border-width: 1px;
  padding: 8px;
  border-radius: 4px;
  border-color: transparent;
  flex: 1 0 0;

  &[type="submit"] {
    background-color: #3298dc;
    color: white;
    &:hover {
      background-color: #2793da;
    }
  }

  &.new {
    background-color: lightgray;
    color: black;
    &:hover {
      background-color: #cccccc;
    }
  }
}

input,
button {
  margin: 16px 8px;
}

.container {
  width: 720px;
  margin: 0 auto;
}

.box {
  border-radius: 6px;
  box-shadow: 0 0.5em 1em -0.125em rgba(10, 10, 10, 0.1),
    0 0px 0 1px rgba(10, 10, 10, 0.02);
  color: #4a4a4a;
  display: block;
  padding: 1.25rem;
}

.hidden {
  display: none;
}

.book-field {
  button {
    background-color: #f14668;
    flex: 0 0 0;
    &:hover {
      background-color: #f03a5f;
    }
  }
}
```

Now head back to [original
post](/blog/rails-nested-attributes-dynamic-forms-and-stimulus-js-part-1).
