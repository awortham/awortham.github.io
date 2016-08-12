---
layout: post
comments: true
title: Using FactoryGirl Traits To Streamline Your Specs
---

-What is FactoryGirl?-

FactoryGirl is a fixtures replacement gem that works beautifully with RSpec and is tremendously helpful in
efficiently setting up your test data for a well tested app. My goal is not to give an explanation of all
the ins and outs of FactoryGirl, so if you are interested in basic setup check it out [here](https://github.com/thoughtbot/factory_girl).

-The Problem-

FactoryGirl is very easy to get up and running. However, the one thing that I struggled with initially was
getting the associations setup. Let's look at an example with a car. Here is a basic way to call the factory
for the car and the defined factory.

```ruby
car = create(:car)

FactoryGirl.define do
  factory :car do
    color "red"
    make "Ford"
    model "F-150"
  end
end
```

Now I want to change the color for my specific spec that I am writing.

```ruby
car = create(:car, color: "blue")
```

This overrides the default color of red that was defined in the factory.
Now what if we want to define some associations? If you have a one to many such as a car has one insurance provider
you can do it like this:

```ruby
FactoryGirl.define do
  factory :car do
    color "red"
    make "Ford"
    model "F-150"
    insurance_provider create(:provider)
  end
end
```

This can be a bad idea because now you are hitting two database tables per spec that calls this factory whether
you need it or not. This also does not work with many to many relationships. You will receive an error.

-Solution-

The simple way to solve this and the way that I took when I first began was to simply assign them after creating
the car. Lets assign some drivers for this car.

```ruby
car = create(:car)

car.drivers << create(:driver)
```

This works for a small test, however as my code base continues to grow, the car starts having all sorts of relations
that may need to be supplied. Soon my specs began to look like this:

```ruby
before do
  car = create(:car)

  2.times do
    car.drivers << create(:driver)
  end

  car.insurance_provider = create(:provider)
  car.accident_history << create(:accident)
  car.sales << create(:sale, price: "45000")
end
```

And on and on. Each time I wanted to write a request spec in which the html was loaded I created something like this.
Then I would write a new model level spec for a new query and would create a new car with all the relations to query
against. This is obviously too much repetitive setup.

-Traits-

This is where traits come in. Traits allow you to do some thorough setup inside your factories file and simply
reference this. Here is an example:

```ruby
FactoryGirl.define do
  factory :car do
    color "red"
    make "Ford"
    model "F-150"

    trait :supreme do
      after(:create) do |article|
        car.insurance_provider = create(:provider)
        car.drivers << create(:driver)
        car.accidents  << create(:accident)
        car.sales << create(:sale)
      end
    end
  end
end
```

This trait can be called with the following:

```ruby
car = create(:car, :supreme)
```

And make that Ford blue:

```ruby
car = create(:car, :supreme, color: "blue")
```

Now we have a clean and concise spec and we can create all relations we want with a simple command. You can
use traits for all sorts of differing setup options and then re use this trait inside your next spec.

Beautiful right?

Traits are yet another way to keep our code dry. The next time you find yourself writing out the database
relations for your object, consider whether a FactoryGirl trait may be the way to go.

-Final Notes-

Some good resources that I have learned from are Thoughtbot's Upcase tutorials and this [blog post](https://robots.thoughtbot.com/remove-duplication-with-factorygirls-traits) by Thoughtbot

