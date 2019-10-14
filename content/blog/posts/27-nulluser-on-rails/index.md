---
id: 27
date: "2014-08-11"
title: "NullUser On Rails"
---
The Null Object Pattern is a popular software pattern that can be very useful as it can get rid of many nil checks in Ruby. Throughout your application, you may have many clauses like `if current_user.admin?`. If you require everyone who uses your application to log in, you won't have any problems, but if you don't, `NoMethodError: undefined method 'admin?' for nil:NilClass`. So how would you get rid of this error? Well, you could simply wrap your `if current_user.admin?` in a `if current_user` check, but you have to do this everywhere `if current_user.admin?` is called. This can be annoying and time consuming to track down and fix. Luckily, the aforementioned Null Object Pattern can get rid of all these checks. Depending on which authentication system you used, you may have to tweak this method a little bit but in [Monban](https://www.github.com/halogenandtoast/monban):

    class ApplicationController < ActionController::Base
      include Monban::ControllerHelpers

      ...

      def current_user
        super || NullUser.new
      end
    end

    class NullUser
      def admin?
        false
      end
    end

What's happening is in `ApplicationController`, `current_user` is being redefined to either the superclass of `current_user`, or a new instance of `NullUser`, if the superclass returns nil. The reason why this works is because in Ruby, when you include a module in your class, the chain of inheritance actually gets changed so it checks the module first, then the parent object, so feel free to call `super` as much as you want. Now, throughout the application, you won't have to change anything, and you won't get that ugly `NoMethodError` message.