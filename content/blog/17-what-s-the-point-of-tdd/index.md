---
id: 17
date: "2014-07-14"
title: "What's the Point of TDD"
---
At Metis, we haven't been using test driven development to help develop our Ruby on Rails apps. Instead of learning TDD, we've been using [Error Driven Development](http://www.halogenandtoast.com/error-driven-development), the baby brother of test driven development. Error driven development is the method of development where you write code till you don't get an error, and you know which errors you are looking for. There is one main problem with this, in Ruby on Rails, the procedure would be: you make a link, make a route, make a controller, make an action, make a template, and then there are no errors, but you have a blank page, so you would have to know to display the Active Record instance on the page.

With test driven development, this is not the case. You can write tests that simulate all of this behavior that a user would have. An added benefit of this is if you want to upgrade a Gem, you can run your tests and see if they pass, else you can just rollback to the previous version of the Gem.

There are benefits to error driven development as opposed to test driven development, one of which is, it is much more welcoming to newcomers because it is much easier to set up, especially if you want to try and set up your tests to run automatically.