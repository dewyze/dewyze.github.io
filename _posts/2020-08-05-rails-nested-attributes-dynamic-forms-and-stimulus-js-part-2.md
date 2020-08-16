---
layout: post
title:  "Rails Nested Attributes, Dynamic Forms, and StimulusJS Part 2"
date:   2020-08-05
permalink: "/blog/rails-nested-attributes-dynamic-forms-and-stimulus-js-part-2"
tags: rails stimulus frontend
excerpt: Image you have a Store form, and you want to be able to add and delete an unlimited number of books in one form submission. Using StimulusJS and Rails Nested Attributes, this is incredibly simple.
difficulty: medium
exclude_from_feed: true
---


## Adding a Row

Our last step is we want to be able to add new books to our store. This will
mean generating new inputs, but we have to make sure it builds the input with
the appropriate ids so rails submits the attributes correctly.

Our solution will look something like this:

![Add Books](/assets/img/add_books.gif)

If we look at the source code of a rails nested attribute, we get something like
this.

```html
<input type="hidden" value="1" name="store[books_attributes][0][id]" id="store_books_attributes_0_id" />
<div>
  <label for="store_books_attributes_1_title">Title</label>
  <div class="fields">
    <input type="text" value="Flour Water Salt Yeast" name="store[books_attributes][1][title]" id="store_books_attributes_1_title" />
    <button name="button" type="submit" data-action="form#deleteRow" data-index="1">Delete</button>
      <input value="0" data-target="form.destroyInput" type="hidden" name="store[books_attributes][1][_destroy]" id="store_books_attributes_1__destroy" />
  </div>
</div>
```

Note that the inputs have names/ids that look like: `store[books_attributes][0][id]`.

They will appear in the rails controller parameters in an even stranger way. They are not an
array, but a hash:


```ruby
{
  store: {
    books_attributes: {
      "0" => {
        id: 1,
        title: "Flour Water Salt Yeast",
        _destroy: 0,
      },
      "1" => {
        # More book info
      },
    },
  },
}
```

We need to build this input for each row we want to add.

If we were using React, odds are our front-end would already have this as a
component we can use, but we were trying to use just plain `erb` files.

We could try and build it in javascript, but then if we change it on the server,
we would need to change the client as well.

There is another option! We can let the server do it! We can simply create an
endpoint which just returns some html and use stimulus to insert it into our
DOM.

## Let the Server Do the Work

By using the ruby server, we can use our existing code to generate an infinite
number of new rows!

Let's create a controller endpoint for our new fields:

```ruby
# app/controllers/form_fields/store_books_controller.rb

module FormFields
  class StoreBooksController < ApplicationController
    def new # 1
      store = Store.new # 2
      store.books.build # 3

      helpers.form_for(store) do |form| # 4
        form.fields_for :books, child_index: params[:index] do |book_form| # 5
          render partial: "stores/book_fields", locals: { book_form: book_form } # 6
        end
      end
    end
  end
end



# config/routes.rb

resources :stores, only: %i[create show edit update] # 7

namespace :form_fields do # 8
  resources :store_books, only: %i[new]
end
```

Let's walk through this:

1. We will just use a `new` method/route. It is idiomatic that it's a get, so we
   don't need a form to get it.
1. In order to use a [Rails
   FormBuilder](http://api.rubyonrails.org/classes/ActionView/Helpers/FormBuilder.html)
   we need to have an object to base it off of.
1. We need to have a shell of a book, otherwise our nested attributes will have
   no books to render fields for.
1. We can use the
   [`helpers`](http://api.rubyonrails.org/classes/ActionController/Helpers/ClassMethods.html#method-i-helpers)
   to get access to view helpers outside of a view. We can use the
   [`form_for`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_for)
   method to instantiate a FormBuilder for our store.
1. Now we can write as if we are in a view. So we will use the
   [`fields_for`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormBuilder.html#method-i-fields_for)
   method just like we did in our edit page. See below for the explanation of
   `child_index`.
1. And then we just render our partial, as html with the new form builder
   passed in.
1. We need a `create` route because the form builder is dealing with a new
   store, it will not be used. We could pass the store ID and use an update, but
   for now create will suffice.
1. And we add a route for our form fields controller.



### Setting the Index

In part 1, we use the form builders index as the data point for identifying what
row we are working on in the stimulus controllers. Additionally, the index is
used to group together all the of the fields for a given book. Therefore, we
need to ensure we use the next index (or at least a new one) when generated new
fields.

Conveniently, the FormBuilder accepts an option called
[`child_index`](https://github.com/rails/rails/blob/fbe2433be6e052a1acac63c7faf287c52ed3c5ba/actionview/lib/action_view/helpers/form_helper.rb#L1680)
for setting the starting index. In order to use this though, we will need that
passed in from the caller. We can grab it from the params here.

## Back to Stimulus

If we look at the [stimulus
example](https://stimulusjs.org/handbook/working-with-external-resources) for
loading content from an external resource, they refer to their controller as
"content-loader." So we can call ours an "content-loader", but ours is different
in that we need params to be passed as well.

Realistically, you could
abstract that into params-content-loader or just make params optional, but
remember, "Embrace the wrong way". Let's get our stuff working and then we can
optimize.

Let's add a few data points to our view and then build our controller.

```erb
<div id="book-form" data-controller="toggle value content-loader" <%# 1 %>
  data-content-loader-next-index="<%= @store.books.length %>" <%# 2 %>
  data-content-loader-insert-location="beforeend" <%# 3 %>
  data-content-loader-url="/form_fields/store_books/new" <%# 4 %>
  data-toggle-class="hidden" data-value-value="1">
  ...
  <div id="book-fields" data-target="content-loader.container"> <%# 5 %>
    ...
    <div class="fields">
      <%= button_tag "Add Book", class: "new", data: { action: "content-loader#insert" } %> <%# 6 %>
    </div>
    <div class="fields">
      <%= form.button "Save", type: :submit %>
    </div>
  </div>
```

1. We add `content-loader` to our `data-controller` value in our form wrapper.
1. We need to know the next index we want, and since we are 0 indexed, we can
   just use the length of the list of existing books. So for 3 books, the final
   index is 2, and our next index is 3.
1. This comes from the
   [`insertAdjacentHTML`](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentHTML)
   api we will use to add this data in. (We are not using jquery.). We will put
   it before the end of the DOM element we pass in.
1. We need a URL that we will reach out to for new content.
1. Finally, we add our book fields container (that contains all book fields)
   where we will put the new book fields before the end of the container.
1. We add a button that will call the `#insert` method on our new `content-loader` controller.

```javascript
// app/javascript/controllers/content_loader_controller.js

import { Controller } from "stimulus";

export default class extends Controller {
  static targets = ["container"]; // 1

  connect() {
    this.nextIndex = this.data.get("next-index"); // 2
  }

  insert() { // 3
    event.preventDefault(); // 4

    let controller = this; // 5
    let queryString = `?index=${this.nextIndex++}`; // 6

    let url = this.data.get("url") + queryString; // 7
    fetch(url) // 8
      .then((response) => response.text())
      .then((html) => {
        controller.containerTarget.insertAdjacentHTML( // 9
          controller.data.get("insert-location"), // 10
          html // 11
        );
      });
  }
}
```

1. We set our container target.
1. We want to get the index only when we connect, because after we grab it from
   the DOM, we will maintain it as a variable in the controller.
1. We will make an "insert" method.
1. Make sure no form is submitted.
1. This is just so we can reference it later in the `fetch` method where the value
   of `this` changes.
1. We will set the query string we will pass to the URL. We pass the index and
   get the value of nextIndex _before_ we increment it.

   Realistically, we may actually want a `data-content-loader-params` value or
   something that is not specific to an index. But let's get this working before
   we worry about that.
1. We grab the url from the DOM and add our query string.
1. We'll use the
   [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) API for
   making this request. (This involves Promises, which if you aren't familiar
   you can go learn about separately. For now, you can just copy this code.)
1. Once we have a html, we will use that
   [`insertAdjacentHTML`](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentHTML)
   method to add our new content, which is in the `html` variable.
1. We will grab the insert location from our DOM.
1. Finally, we provide our new html.

And then:

![Add Books](/assets/img/add_books.gif)

We didn't have to replicate our templating logic anywhere, and could just reuse
some rails magic and our existing views.
