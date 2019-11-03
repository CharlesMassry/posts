---
id: 38
date: "2014-10-07"
title: "Devise on Rails"
---
With most web apps, you'll want to have some type of authentication system so your users' data is secure. The question then shifts of how to implement user authentication, while you can roll your own, like I described in an earlier [post](/posts/13), it can become a time sink really quickly but fortunately there's a gem for that. There are actually many gems for authentication, in fact, if you look at the [Ruby Toolbox page](https://www.ruby-toolbox.com/categories/rails_authentication), there are 19 to be exact. Topping the charts is the popular gem [Devise](http://devise.plataformatec.com.br/) from our friends over at [Plataformatec](http://plataformatec.com.br/) in Brazil.

Devise provides many features and has 10 different modules such as the Omniauth support. It can however be difficult to set up, and even the documentation explicitly states to look into how authentication works in Rails and provides some links to sources that detail how to roll your own. To get started, add to your Gemfile

    gem "devise"

Next run the Devise generate command

    $ rails generate devise:install

This generates two files, `config/initializers/devise.rb` and `config/locales/devise.en.yml`. `devise.rb` holds different configuration values, such as password length or email validation and `devise.en.yml` holds different Devise messages such as confirmation instructions. Now that you have these files, you can create users. You can call your users anything you want, but if you are just getting started you would probably just want to call them users. To do that you must make sure your database is created and then you can generate your User model with Devise.

    $ rails generate devise User

This generates a couple of files, the migration and the User model. When you check the migration you will see a lot of cool database columns that will be created and auto-tracked, such as from the Trackable module, like `last_sign_in_ip` or `last_sign_in_at`. You can add your own as you please like `username` or `first_name` and `last_name` for example. This however requires some configuration due to the Strong Params in Rails 4. For example, lets add first name and last name support for the users. In our created migration

    class DeviseCreateUsers < ActiveRecord::Migration
      def change
        create_table(:users) do |t|
          ...
          t.string :first_name
          t.string :last_name
          ...
        end
        ...
      end
    end

Now that we've finished the migration

    $ rake db:migrate

To add the new columns to our view forms we must enter into the forms located in `app/views/devise/registrations/new.html.erb`. To even see the Devise created views within your app enter

    $ rails generate devise:views

This creates the Devise views in the `views/devise` directory.

Within the `form_for` block add

    <div>
      <%= f.label :first_name %>
      <%= f.text_field :first_name %>
    </div>
    <div>
      <%= f.label :last_name %>
      <%= f.text_field :last_name %>
    </div>

When we submit this form however, it will not work because of the Strong Parameters I mentioned earlier, which really highlights what I think is one of the main problems with Devise, as opposed to a simpler gem like [Monban](https://github.com/halogenandtoast/monban), it hides too much of what is going on which makes modification difficult. To add the first name and last name fields, we won't create a Devise controller like we did with the views, instead we will modify the `ApplicationController` with a `before_action` to modify a hidden Devise controller

    class ApplicationController < ActionController::Base
      ...
      before_action :configure_permitted_parameters, if: :devise_controller?

      protected

      def configure_permitted_parameters
        devise_parameter_sanitizer.for(:sign_up) << [:first_name, :last_name]
      end
    end

You can create your own controller using the Devise controller generator which you might want to switch to if your application grows and needs more customization or different types of users with different sign in processes. But for now, your User model is ready to roll with Devise and any extra fields you care to add to users.