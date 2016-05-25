---
layout: post
comments: true
title: Using Draper to Clean up Views
---

-Rails Decorators Using Draper Gem-

There are several ways to clean up your views and many helper methods available to Rails.
The Draper gem is rather like a wrapper around your Rails models. The decorator is used
to encapsulate view logic, but can be tested like a model.


Begin by adding the draper gem to you gem file.

```ruby
gem 'draper'
```

Then run bundle in your console.

Lets say we have a collection of three articles with a published date that we want to display.
First we need to generate the decorator that we will need for the article objects. Draper
has a generator built in that makes this super simple.

```ruby
rails g decorator article
```

This command will generate a folder called decorators inside the app directory and
a file called article_decorator.rb.

Next we will need to 'decorate' the object or collection of objects.

In our controller we make our query and decorate it:

```ruby
@articles = Article.publishable.first(3).decorate
```

This works when we call it on the active record relation, but would not work if your
collection were an array of article objects.


Now, no feature is complete without a good spec, so lets map out what it is we are
looking for. I like to create a decorator directory in the spec directory and create
a spec file called article_decorator_spec.rb.

```ruby
require "rails_helper"

RSpec.describe ArticleDecorator, type: :model do
  it "formats the published date" do
    article = create(:article, published_date: DateTime.new(2014,2,12,6,30)).decorate

    expect(article.format_published_date).to eq "Wednesday, February 12, 2014  6:30 AM"
  end
end
```

Next to make our test pass.

```ruby
  def format_published_date
    modified_date.strftime("%A, %B %d, %Y %l:%M %p")
  end
```

That should get our spec to green. Now lets go ahead and implement it into the views.

```ruby
  <% @articles.each do |article| %>
    <%= article.format_published_date %>
  <% end %>
```

And there it is! We have now implemented a decorator using the Draper gem and moved
the formatting logic out of the view and into the decorator class.

