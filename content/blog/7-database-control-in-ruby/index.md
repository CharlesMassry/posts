---
id: 7
date: "2014-06-26"
title: "Database Control in Ruby"
---
Active Record is the preferred method of interacting with a database. Being a Ruby Gem independent of Ruby on Rails, Active Record can work with other frameworks such as Sinatra. [Active Record](https://github.com/rails/rails/tree/master/activerecord) is simple to setup and the commands are not confusing at all.

    SELECT galleries.*
    FROM galleries
    WHERE (name = 'Cats');

This would return all database entries in the galleries table that have name equal to ‘Cats’. This is good, but it is kind of a long statement to type out and you can’t interact with it much by just using SQL. For example, putting it on the web would be a lot more difficult.

    Gallery.where("name = 'Cats'")

This gives you the same result as the previous SQL statement, except you can now interact with it in Ruby, and thus Sinatra and put it on the web.

But Charlie, that’s all well and good but how do you use a `JOIN` command with Active Record, you ask? Well if you want to join a galleries to an images table you would write:

    class Image < ActiveRecord::Base
    end

    class Gallery < ActiveRecord::Base
      has_many :images
    end  

The has many method returns all of the images that are associated with the gallery where the Foreign ID of the images is equal to the ID of the gallery, only when you call the images getter method. Active Record automagically creates all these different methods for the database, such as all getter and setter attribute methods, just never use the ID or Foreign ID setter methods otherwise you’ll ruin your database.

    @gallery = Gallery.find_by(name: “Cats”)` 

This Ruby query would return `#<Gallery id: 1, name: "Cats", description: "Funky cats”>`. If you then wrote `@images = @gallery.images`, the return value would be a long listing of every image that matches its Foreign ID to the ID of the gallery. You can then iterate through the `@images` instance variable and post each image to the web using Sinatra and an ERB template.

    @galleries = Gallery.where("name = 'Cats’”)
    @images = @galleries.map { |gallery| gallery.images }
    @images.flatten!
    @images.each { |image| image.url }

This would return a list of all the URLs that correspond to the Cats gallery. The reason for this seemingly complex code is because `.where(“name = ‘Cats’”)` returns an array that needs to be iterated through to find the images and then flatten the array that `.map` returns. You are then left with one array and can iterate through that array to return each image URL.

In any event, just make sure that your casing is correct.