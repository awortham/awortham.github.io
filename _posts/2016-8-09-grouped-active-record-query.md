---
layout: post
comments: true
title: Active Record Grouped and Counted Query
---

Recently, I came across an interesting problem. A few quick Google searches revealed that I am not the first
person to run into this. But I also discovered that people generally thought this was much easier to solve
using straight SQL rather than ActiveRecord.

### Most Popular Articles

My challenge was to find the most popular or most visited articles and display these articles in a widget
at the bottom of the page or in a sidebar.

### Database Setup

I created both an articles table and a trendings table. The separate trendings table allowed me to set
up for trendings tags also and not just trending articles, but we will focus on the articles here. The
migration for trendings looks like this:

```ruby
class CreateTrendings < ActiveRecord::Migration
  def change
    create_table :trendings do |t|
      t.integer  :article_id, default: 0
      t.integer  :tag_id,     default: 0
      t.datetime :watched_at

      t.timestamps null: false
     end
   end
end
```

My goal here is to query the recently trending articles, therefore the watched_at column will allow us
to target the most recent trendings.

### Models

Next we need to make sure the relationships are set up correctly. A couple of specs and we are up and running.

```ruby
it "has many trendings" do
  article = create(:article, trendings: [create(:trending)])

  expect(article.trendings).to_not be_empty
end

class Article
  has_many :trendings
end
```

```ruby
it "belongs to articles" do
  article  = create(:article)
  trending = create(:trending, article: article)

  expect(trending.article).to eq article
end

class Trending
  belongs_to :article
end
```

Then I wrote a spec for the trending class that would return all trendings that had an article specified. This
ensures that we do not return trendings that were meant for tags.

```ruby
  it "queries by article id" do
    create(:article)
    2.times { create(:trending, article_id: 1) }

    tag_trending = create(:trending, article_id: 0)

    expect(Trending.popular_articles.count.values).to include 2
    expect(Trending.popular_articles).to_not include tag_trending
  end
```

Now to get this to pass. I want a query that groups the records.

```ruby
scope :popular_articles, -> { group(:article_id).where.not(article_id: 0) }
```

This query tells me that I have two trendings with the same article id.

Now for the good stuff. Back to the article model spec. First I created a before block that runs our setup.
This block creates articles and assigns trendings to them.

```ruby
describe "most popular" do

  before do
    10.times do
      create(:article, trendings: [create(:trending)])
    end
    Article.all.each do |a|
      a.id.times do
        create(:trending, article_id: a.id)
      end
    end
  end

  it "returns most popular articles" do
    article_without_trending = create(:article)
    articles = Article.popular

    expect(articles.first).to eq Article.last(2).first
    expect(articles).to_not include article_without_trending
  end

end
```

And the corresponding query to make this spec pass.

```ruby
scope :popular, -> (limit=10) { joins(:trendings).merge(Trending.popular_articles).order("count(articles.id) DESC").limit(limit) }
```

This query joins with the trendings table, grabs our popular articles method from and then orders them by the number
of times the article id shows up in that collection. Cool right? I also gave it a default limit of 10. This
way if I want differing numbers of records on different pages it is very easy to do so.

### Finishing Touches

What if I want to know the number of trendings that were for any given article? I used a regular joins here,
so without an N+1 query I cannot access this number. Here are some final adjustments to give us access to
these records.

```ruby
scope :popular, -> (limit=10) { includes(:trendings).merge(Trending.popular_articles).order("count(articles.id) DESC").limit(limit).references(:trendings) }
```

Note: You must add the 'references' at the end if you want to do it this way. Now you can access the number of
trendings per article like this:

```ruby
def trending_count
  trendings.length
end
```

### Final Notes

In this example we were able to group trendings inside the trendings table, count those groups and order the
popular articles from the article model and also make the number of trendings available without an N+1 query.
All with one well written query in Active Record. And we never touched pure SQL.

Cheers!

