---
id: 10
date: "2014-07-01"
title: "MVC...R?"
---
Ruby on Rails follows the popular Model, View, Controller paradigm. One thing of note about this is Rails’s separation of the routes from the controller. While frameworks like Sinatra and Express have the routes and controllers as one, Ruby on Rails separates this into separate files. This can be confusing but I feel that it leads to cleaner code. Consider the show page.

The route: `get "/galleries/:id" => "galleries#show"`

The controller:  
  
    class GalleriesController < ApplicationController
      def show
        @gallery = Gallery.find(params[:id])
      end
    end

In Sinatra identical code would look like:  
    
    get "/galleries/:id" do
      @gallery = Gallery.find(params[:id])
    end

While there doesn’t really seem to be much of a benefit to doing this with Rails as there is objectively more keystrokes that must be typed, Rails does have some shortcuts that use the “Convention over Configuration” principle. This means that you don’t have to do trivial things that Rails assumes you are going to do, so you would think there would be a shortcut in Rails to create all 7 HTTP requests for a given model. According to the Rails documentation, you can simply type `resources :galleries` and Rails will create all of the 7 HTTP requests needed for the CRUD operations, along with methods that point to the GET paths, called path helpers, such as `gallery_path(gallery.id)`. This would be useful in your views when you type in a link\_to helper, you can type in `` instead of ``. This creates much cleaner looking code without using messy strings. Also this illustrates how important the `config/routes.rb` is in your Rails app, even though it is not in your app directory like your models, views, and controllers are. 