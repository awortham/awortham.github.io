---
layout: post
comments: true
title: Using Draper to Clean up Views
---

-Rails Decorators Using Draper Gem-

There are several ways to clean up your views and many helper methods available to Rails. The Draper gem allows you to focus these helpers into one view so they don't conflict and can clean up your views, which gives you concise beautiful code.

The steps in this blog are based on Rails version 4.1.6 and Ruby version 2.1.3

Begin by adding the draper gem to you gem file.









Then run bundle in your console.







I am adding this into some budgeting software that I am writing, so translate what you see from the controller to your own project. This is what I had to begin with:














In order to set up for the decorator I will change it to be this:
















This allows you to still insert all the accounts in the database, but now you have wrapped each one in the decorator.
Now, if you try to load the index action it should blow up and give you this error on your local server:
















The next step is to generate the decorator. To do this run 'rails g decorator account' in your console. This should generate a folder called 'decorators' and a file named 'account_decorator.rb'. Now restart your console and you should see your index page as you normally would.

In my index view page I have inserted the line <%= account.created_at %> which displays this on the page.








This is not very readable. The traditional Rails way to fix this is to add something like this to your view page:

<%= account.created_at.strftime("%b %e, %l:%M %p") %>

This changes the display to this:










From the user side this is fine, however my goal is to get the view page cleaned up behind the scenes. So here is where the decorator comes in.

Open your account_decorator.rb file and write a method called format_created_at. The name is not important so long as you call the same method name in your view template. Here is what mine looks like:



Then change your view so that it calls this method rather than the original created at method.

<%= account.format_created_at %>

Bravo!!

You have now implemented a helper method using decorators from the draper gem. The output on your page should be the same from the user standpoint but your code has been cleaned up and you pushed the formatting logic out of the template and into the decorator class.

Twitter: https://twitter.com/WorthamAaron
Github: https://github.com/awortham
