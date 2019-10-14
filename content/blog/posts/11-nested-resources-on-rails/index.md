---
id: 11
date: "2014-07-02"
title: "Nested Resources on Rails"
---
There are a few key things you must remember when working with nested resources in Ruby on Rails. The first thing to do is edit your routes file.

    resources :galleries do
      resources :images
    end 

This creates all of those routes with the HTTP verbs that point to the controller#action and creates the URL helpers. Much less typing than Sinatra.

The next thing to do is work through the same process you would do without nested resources with some minor exceptions. For example, you have to create the controller with the actions and then the views. One thing to remember is when you make a form in a nested resource, you have to include both resources in the controller.

    def new
      @gallery = Gallery.find(params[:gallery_id])
      @image = Image.new
    end 

This is because `#new` needs to have both gallery and image objects to pass into the `form_for` helper method. `form_for` needs the information about the parent resource in order to create a hidden field for the foreign key.

    <%= form_for [@gallery, @image] do |form| %>
      ...
    <% end %>

`[@gallery, @image]` gives it the new gallery image post path, which Rails knows that it is the path that you want because you are coming from the GET new gallery image path. If this was in the edit method, then you would pass the same arguments in, except for you would pass in an existing image instance in the images controller and not a new one.