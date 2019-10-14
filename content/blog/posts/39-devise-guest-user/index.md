---
id: 39
date: "2014-10-09"
title: "Devise Guest User"
---
Using Devise in Ruby on Rails can be very convenient for authentication, but there are some necessary workarounds that you must be aware of, specifically when you introduce the Null Object Pattern. I had described in a [previous post](/posts/27) how to use the Null Object Pattern with [Monban](https://github.com/halogenandtoast/monban), which was very simple. Using this pattern with [Devise](http://devise.plataformatec.com.br/) however is not so simple.

If you follow the Devise guide on [how to create a guest user](https://github.com/plataformatec/devise/wiki/How-To:-Create-a-guest-user) it encourages you to create a guest user in the database, which can be useful for certain circumstances, like when you need your Guest User transfer over information when they actually sign up, but if not, we can just create a Null User.  

You might have in your app a bunch of checks like `<% if current_user.admin %>` and if you allow guest users, with Devise, you need to either wrap everything in a `<% if user_signed_in? %>`, or create a Guest User. Having a Guest User object with all the same method names return false can be very helpful for this situation, as you can just create a dummy method everytime you get a `NoMethodError: undefined method admin? for nil:NilClass`. To begin, we would write in our `ApplicationController`

    def current_user
      super || GuestUser.new
    end

There is still a Devise gotcha here. The method `user_signed_in?` only returns a falsy value if `current_user` is falsy. Since a Guest User is not nil it won't return nil. What we must do is create `User#signed_in?` and `GuestUser#signed_in?` methods that respond with true and false respectively. and use that for our checks, which will look like

    <% if current_user.signed_in? %>
      <% link_to "Sign out", destroy_user_session_path %>
    <% end %>

This will help keep your code [SOLID](http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) and you won't even touch your database.