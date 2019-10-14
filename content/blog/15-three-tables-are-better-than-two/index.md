---
id: 15
date: "2014-07-10"
title: "Three Tables Are Better Than Two"
---
In SQL, you can't have two tables that have each other's foreign key, you must use a third table to hold the foreign keys of the relationship. There are two ways to do this when using Ruby on Rails.

The first and most common way is to use the `has_many :through` Active Record Association. You must create the third table and create an `ActiveRecord::Base` model. This is beneficial because you can add to it later if you need to.

The other, less popular way is to use the `has_and_belongs_to_many` Active Record Association. This requires just a database migration and not a new model. The main problem with this is you cannot customize the association with any business logic, such as validations and callbacks, so it doesn't lend itself to future-proofing code. Either method however, will allow you to call a getter method on the other tables objects it is associated with. 