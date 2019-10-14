---
id: 13
date: "2014-07-07"
title: "Cookie Monster Would Not Be Impressed By These Cookies"
---
In every Internet Browser, there are what are called cookies. These are a way of overriding the fact that HTTP is a stateless protocol, meaning that each HTTP request stands on its own. Without cookies, you would have to manually type in your login information with every HTTP request, which would make the Internet a lot less fun. Cookies store this information in your browser so you don't have to and send this information with every request.  

Although this seems pretty simple, the web developer must include special code to tell your browser what to store as cookies. For example, in Ruby on Rails, a new `SessionsController` must be created to manage user sessions, and instead of creating a `Session` model to store information in the database, the information must be stored in the user's browser as an encrypted string. this information must also be accessed when the user signs out.

    class SessionsController < ApplicationController
      def new
      end

      def create
        user = User.find_by(email: params[:session][:email])
        cookies.signed[:user_id] = user.id
        redirect_to galleries_path
      end

        def destroy
          cookies.delete(:user_id)
          redirect_to galleries_path
        end
      end

What's happening here is the new method just directs the user to the sign in form. Then, once the user signs in, it finds the user by the email address, which is stored in the session hash inside of the params hash as an encrypted string. This is then checked to see if the `user_id` is the same as the decrypted string. When signing out, `destroy` is called and the `user_id` field in the cookies is deleted.

    class ApplicationController < ActionController::Base
      def current_user
        @user ||= User.find_by(id: cookies.signed[:user_id])
      end
      helper_method :current_user

      def signed_in?
        current_user
      end
      helper_method :signed_in?
    end

Here we have the application controller, which uses the cookies in the browser to tell the views which user is currently signed in, and if a user is signed in. It is helpful to have these two methods even though they do the same thing because you can use `current_user` to display the name of the current user, while you can use `signed_in?` for conditionals in `.erb`. Being the `ApplicationController`, it can be called from any view application wide by including `helper_method`, this leads to keeping your code DRY.