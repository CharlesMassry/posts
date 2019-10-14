---
id: 8
date: "2014-06-27"
title: "HTTP Requests in Sinatra"
---
In HTTP, there are four common request actions that are used by servers, GET, POST, PUT, and DELETE. This maps to CRUD, Create, Read, Update, and Delete. Unfortunately, the browsers can only send the GET and POST requests and not PUT and DELETE. To combat this, the server must fake the requests by using a hidden field to change the type of request in the browser. Consider the PUT request. The server file would have to have a GET method to a new page that has a form for updating, and that new page must have a form that uses the PUT method.

  

    # server.rb 
    get "/galleries/:id/edit" do
      @gallery = Gallery.find(params[:id])
      erb :edit_gallery
    end

    put "/galleries/:id" do
      gallery = Gallery.find(params[:id])
      gallery.update(params[:gallery])
      redirect to("/galleries/#{params[:id]}")
    end

    views/edit_gallery.erb
    <form action="/galleries/<%= @gallery.id %>" method="POST">
      <input type="hidden" name="_method" value="put">
      ...
    </form>

While you have to do the same for the POST request as far as the server.rb file is concerned, you must add that second line in the edit\_gallery.erb file. One thing of note is the `name=“_method” value=“put”` snippet. This is how Sinatra determines that you wanted it to be a PUT request and not a POST request and looks for a route that matches a put method.

The DELETE method is easier as you just need to create a delete input button as opposed to a creating a new GET request and new erb file. Just don’t forget to add the hidden input field.