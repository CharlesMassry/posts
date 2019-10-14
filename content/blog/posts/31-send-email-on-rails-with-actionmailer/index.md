---
id: 31
date: "2014-09-03"
title: "Send Email on Rails with ActionMailer"
---
In Ruby on Rails, ActionMailer allows you to email users from your server with processed Ruby code, which can serve any number of uses. There is some configuration required on the back-end to actually set up ActionMailer to actually send mail, but you can fake it for development purposes. There are certain things to be aware of when using ActionMailer. Specifically, you must define instance methods in the mailer class and you must call the instance methods on the mailer class when attempting to send the mail.

  

    # app/mailers/user_mailer.rb
    class UserMailer < ActionMailer::Base
      ...
      def welcome(user)
        @user = user
        mail to: @user.email,
          subject: "Welcome to My Fabulous Web App!"
      end
      ...
    end

    # app/controllers/users_controller.rb
    class UsersController < ApplicationController
      ...
      def create
        user = sign_up(user_params)

        if user.valid?
          UserMailer.welcome(user).deliver
          redirect_to user
        else
          render :new
        end
      end
      ...
    end

    # app/views/user_mailer/welcome.html.erb
    <h1>Welcome <%= @user.email %> to my <em>Fabulous</em> Web App.</h1>
    <p>Visit <%= link_to "my site", root_url %> as much as you can.</p>

There are quite a few things going on here, not the least of which is the `UserMailer.welcome(user)` call. In this particular snippet of code, the user is created and sent a welcoming email. The view is also noteworthy, as it is being called by the mailer, which functions as a sort of controller and renders its view. Another thing about the view is when you include links, you must use url helper instead of path helper as the user's computer has no base reference of the path to the site, and so, you must configure Rails to know what the domain name it is being hosted at.  

Testing emails can be difficult as you have to check that the mail got sent, and that the view contains the appropriate text. Rails provides a method for testing if mail was sent, which is `ActionMailer::Base.deliveries`. This returns an array of the different mail that was sent, but testing the views won't be so easy. Luckily there's a gem for that. The [email\_spec](https://github.com/bmabey/email-spec) gem makes testing the email contents really simple, so you can write your expectations clearly about the mailer and not worry about your tests failing but your code actually working.
