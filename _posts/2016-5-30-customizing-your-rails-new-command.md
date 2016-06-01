---
layout: post
comments: true
title: Customizing your Rails new Command
---

Take care of boilerplate with a few customizations

-The Problem-

There comes a day in every Rails developer's working life that things begin to settle
into place. You have settled on certain gems that you know and prefer to use with
each application that you build. Your test suite is setup the same each time and
you have adjusted the generators to only create the files that you will use. The only
problem with all this is that now all of this setup has become boilerplate code
that you can write in your sleep and yet still shoots half a day or more of setup
for a new application before you can really start to 'build'. With this in mind, lets
look at how we can make all of this happen automatically, saving both you and your
company hours of time each time you launch a new project.

-The Solution-

When I started down this path I looked at a couple of possibilities. Thoughtbot
built a gem called Suspenders for their startup. I tried it, but they use different
gems and have a rather different setup than we do where I work. I considered building
my own version of it, but I really did not want to upkeep a gem at this time. I really
wanted something more lightweight. If you find yourself using the same database migrations
and even some view templates across your platforms, you could look into building
a Rails engine. I've used this for specific things, but this is really for solving
something different than we are talking about now. I settled on customizing what
happens when I run the rails new <app_name> command. This keeps things localized,
easy to maintain and I can quickly make a change and use it all in just a couple
minutes.

```ruby
rails new my_app
```
-Flags-

The easiest thing to change is to add a flag to your rails new command.
Here is an example:

```ruby
rails new my_app --skip-test-unit
```

This flag will tell Rails to skip installing the test unit testing framework, presumably
so that you can use another test framework, such as RSpec. There are a variety of
flags that can be passed to this command, but remembering them all when I needed them
seemed a hassle and I was always looking each one up. There must be a better way.
If you do not have a .railsrc file in your home directory, then go ahead and create
one.

```ruby
touch .railsrc
```

Inside this file, we can pass as many flags as we want. Here is what mine looks
like currently:

```ruby
--database=mysql
--skip-turbolinks
--skip-test-unit
--skip-bundle
environment "config.generators {|g| g.helper false\n g.assets false\n end}"
```

I'll touch on each of these briefly. The database line sets our default database to
MySQL rather than SQLite. The next line skips turbolinks right from the get go rather
than removing this gem and all references to it manually. I have skipped the test unit
because I prefer to use RSpec as my test framework. The next line skips bundle. I added
this line because I want to add my own gems and then bundle later. The last line
configures the standard rails generators to skip generating the helper and asset
files.

-Next Steps-

Now the exciting part begins. Rails templates allow you to add gems, run commands,
generate files and even make git commits as you see fit. Here is the line in my
.railsrc that I've added directly after the previous lines.

```ruby
-m ~/app_template.rb
```

Here is a sample of my app_template.

```ruby
gem_group :development, :test do
  gem 'rspec-rails'
  gem 'spring-commands-rspec'
  gem 'factory_girl_rails'
  gem 'pry'
  gem 'pry-doc'
end

gem_group :test do
  gem 'simplecov'
  gem 'capybara'
  gem 'launchy'
  gem 'database_cleaner'
  gem 'shoulda-matchers'
  gem 'webmock'
  gem 'rack_session_access'
end
```

This will install RSpec gem along with the support gems that I use in my day
to day building and testing. Now lets coordinate it so that when we run our rails
new <app_name> command it also runs the RSpec generate commands.

```ruby
run 'bundle install'
generate 'rspec:install'
run 'bundle exec spring binstub rspec'
git :init
git add: "."
git commit: "-m 'Initial Commit'"
```

-Conclusion-

This bundles all of our gems into the app, install RSpec and even sets up Spring
commands. Lastly, we have made our initial Git commit so that we are ready to go.
This is just a sample of commands and customizations that you can add to your setup
to streamline your setup and get apps off the ground quickly and efficiently. And
all this is rolled up into the command we started with:

```ruby
rails new my_app
```
